Singularity deployment tutorial
 ==============================

# Installation
 - [**Singularity**](http://singularity.lbl.gov/install-linux)
 - [_Docker_](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) (only for getting the rbbt-basic image during the bootstrapping)

# Bootstrapping
1. `singularity create -F -s 2048 rbbt-basic.img` (although it could be less space)
2. `sudo singularity bootstrap -f rbbt-basic.img rbbt-singularity-basic`

# How does it work?
rbbt-singularity-basic container requires (or creates them by default if don't exist) in working directory the `singularity_cache` folder which contains:
- `.rbbt`: all resources, databases, jobs, temporal files, etc... are placed here (this folder or any of its subfolders can be replaced by a symbolic link to and existing workflows directory)
- `workflows`: all workflows are placed here (this folder can be replaced by a symbolic link to an existing workflows directory)
- `.ruby_inline`: for getting the right behavior of RubyInline gem this folder is mandatory

These folders have to be symbolically linked to the original /home/rbbt directory so a init script is embedded to the container during the bootstrapping which wraps the original rbbt command and does all work every time the Singularity container is run.

The bootstrapped image can be removed, created and bootstrapping again without lost all job during the previous bootstrapping because the existing singularity_cache is retained

# Usage
## Shell

`singularity shell --writable -e rbbt-basic.img`

## Exec
### Real world example

`singularity exec --writable -e rbbt-basic.img rbbt workflow install Appris`

`singularity exec --writable -e rbbt-basic.img rbbt workflow install Sequence`

    singularity exec --writable -e rbbt-basic.img rbbt workflow task Sequence mutated_isoforms_fast -m data/mis.txt /bin/bash

where data/mis.txt is found together the Singularity image and provided in this repository as a toy example. A data/ folder to place all input data is recommend but not required

# Why is the bootstrapping needed?

The base image for Singularity is taken from Docker but have to be customized during the bootstrapping process because:
- Singularity overlaps /home and /home/rbbt is hidden for all users other than root.
- Singularity takes the outside user to inside so it can never be predicted (without be hardcoded)
- Singularity can use bind points but are forbidden in HPC environments along with others constrains
