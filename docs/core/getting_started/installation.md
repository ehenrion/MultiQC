---
title: Installation
description: How to install MultiQC on your system
order: 2
---

# Installing MultiQC

MultiQC is written in Python and can be installed in a number of ways.
Which method you should use depends on how you're using MultiQC and how familiar you are with the Python ecosystem.

:::info

This page can be a little overwhelming. If in doubt:

- Running MultiQC in a pipeline? Use [Docker](#docker) or [Singularity](#singularity).
- Running MultiQC locally? Use [Pip](#pip) or [Conda](#conda).

:::

:::tip{title="TL;DR;"}

Know what you're doing with this kind of thing? Here's a quick reference:

#### Pip

```bash
pip install multiqc
```

#### Pip (dev version)

```bash
pip install --upgrade --force-reinstall git+https://github.com/ewels/MultiQC.git
```

#### Conda

```bash
conda install multiqc
```

#### Docker

```bash
docker run -t -v `pwd`:`pwd` -w `pwd` ewels/multiqc multiqc .
```

:::

## Python

Most people running MultiQC manually install it directly into their local Python environment.

MultiQC requires Python version 3.6 or above.

### System Python

Python comes installed on most operating systems. You can install MultiQC directly here, but it is _not_ recommended. This often causes problems and it's a little risky to mess with it.

:::danger
If you find yourself prepending `sudo` to any MultiQC commands, take a step back and think about Python virtual environments / conda instead (see below).
:::

### Python with Conda

To see if you have python installed, run `python --version` on the command line.
MultiQC needs Python version 2.7+, 3.4+ or 3.5+.

We recommend using virtual environments to manage your Python installation.
Our favourite is _conda_, a cross-platform tool to manage Python environments.
You can installation instructions for Miniconda
[here](https://docs.conda.io/en/latest/miniconda.html).

Once conda is installed, you can create a Python environment with the following commands:

```bash
conda create --name py3.7 python=3.7
conda activate py3.7
```

You'll want to add the `conda activate py3.7` line to your `.bashrc` file so
that the environment is loaded every time you load the terminal.

### Using a specific python interpreter

If you prefer, you can also run MultiQC with a specific python interpreter.
The command line usage and flags are then exactly the same as if you ran just `multiqc`.

For example:

```bash
python -m multiqc .
python3 -m multiqc .
~/my_env/bin/python -m multiqc .
```

## Local installation

There are a few different ways to install MultiQC into your local Python environment:

### Conda

MultiQC is available on [BioConda](https://bioconda.github.io/).
First, [configure your conda channels](https://bioconda.github.io/#usage) then install with:

```bash
conda install multiqc
```

:::warning
The order of conda channels is important. Do not use `-c bioconda` in the installation command, weird stuff can happen (like only being able to install very old versions).
:::

### Pip

This is the easiest way to install MultiQC. `pip` is the package manager for the Python Package Manager. It comes bundled with recent versions of Python,
otherwise you can find installation instructions [here](http://pip.readthedocs.org/en/stable installing/).

You can install MultiQC from [PyPI](https://pypi.python.org/pypi/multiqc) as follows:

```bash
pip install multiqc
```

Use the `--upgrade` flag to update to the latest version.

If you would like the development version, the command is:

```bash
pip install git+https://github.com/ewels/MultiQC.git
```

To update the dev version between releases, use `--upgrade --force-reinstall`. This is needed as the version number isn't changing.

If you have problems with read-only directories, you can install to
your home directory with the `--user` parameter:

```bash
pip install --user multiqc
```

### Manual

If you'd rather not use either of these tools, you can clone the code and install the code yourself:

```bash
git clone https://github.com/ewels/MultiQC.git
cd MultiQC
pip install .
```

This will fetch the latest development code. To update to the latest changes, use `git pull`.

`git` not installed? No problem - just download the flat files:

```bash
curl -LOk https://github.com/ewels/MultiQC/archive/master.zip
unzip master.zip
cd MultiQC-master
pip install .
```

## Within a Python script

You can import and run MultiQC from within a Python script, using
the `multiqc.run()` function as follows:

```python
import multiqc
multiqc.run("/path/to/dir")
```

## Docker

A Docker container is provided on Docker Hub called `ewels/multiqc`.
It's based on an `python-slim` base image to give the smallest image size possible.

To use, call the `docker run` with your current working directory mounted as a volume and working directory. Then just specify the MultiQC command at the end as usual:

```bash
docker run -t -v `pwd`:`pwd` -w `pwd` ewels/multiqc multiqc .
```

- `-t`: Runs docker with a pseudo-tty, for nice terminal colours
- `-v`: Mounts the current working directory into the container
- `-w`: Sets the working directory in the container as your local working directory

You can specify additional MultiQC parameters as normal at the end of the command:

```bash
docker run -t -v `pwd`:`pwd` -w `pwd` ewels/multiqc multiqc . --title "My amazing report" -b "This was made with docker"
```

By default, docker will use the `:latest` tag. For MultiQC, this is set to be the most recent release.
To use the most recent development code, use `ewels/multiqc:dev`.
You can also specify specific versions, eg: `ewels/multiqc:1.9`.

Note that all files on the command line (eg. config files) must also be mounted in the docker container to be accessible.
For more help, look into [the Docker documentation](https://docs.docker.com/engine/reference/commandline/run/).

:::tip{title="Docker bash alias"}

The above base command is a little verbose, so if you are using this a lot it may be worth adding the following bash alias to your `~/.bashrc` file:

```bash
alias multiqc="docker run -tv `pwd`:`pwd` -w `pwd` ewels/multiqc"
```

Once applied (first log out and in again) you can then just use the `multiqc` command instead:

```bash
multiqc .
```

:::

## Singularity

Although there is no dedicated Singularity image available for MultiQC, you can use the above Docker container.

First, build a singularity container image from the docker image (change `1.14` to the current MultiQC version):

```bash
singularity build multiqc-1.14.sif docker://ewels/multiqc:1.14
```

Then, use `singularity run` to run the image with the normal MultiQC arguments:

```bash
singularity run multiqc-1.14.sif my_results/ --title "Report made using Singularity"
```

:::info{title="Import errors with Singularity"}

Sometimes, Singularity can be over-ambitious with sharing file paths which can result in the Python environment in your local system interacting with Python inside the image. This can give rise to `ImportError` errors for `numpy` and other packages.

The giveaway for when this is the problem is that traceback will list python package paths which are on your system and look different that of MultiQC inside the container (eg. `/usr/lib/python3.8/site-packages/multiqc/`).

To fix this, run the command `export PYTHONNOUSERSITE=1` before running MultiQC. This variable [tells Python](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONNOUSERSITE) not to add site-packages to the system path when loading, which should avoid the conflicts.
:::

## Spack

MultiQC [is available](https://spack.readthedocs.io/en/latest/package_list.html#py-multiqc) on [Spack](https://spack.io/) as `py-multiqc`:

```
spack install py-multiqc
```

## Nix

If you're using the [nix package manager](https://nixos.org/download.html#download-nixm) with [flakes](https://nixos.wiki/wiki/Flakes) enabled, you can
run `nix develop`in the MultiQC repository to enter a shell
with required dependencies. To build MultiQC, run `nix build`.

## Galaxy

### On the main Galaxy instance

The easiest and fast manner to use MutliQC is to use the [usegalaxy.org](https://usegalaxy.org/) main Galaxy instance where you will find [MultiQC Galaxy tool](https://usegalaxy.org/?tool_id=toolshed.g2.bx.psu.edu%2Frepos%2Fengineson%2Fmultiqc%2Fmultiqc%2F1.0.0.0&version=1.0.0.0&__identifer=2sjdq8d9r3l) under the _NGS: QC and manipualtion_ tool panel section.

### On your instance

You can install MultiQC on your own Galaxy instance through your Galaxy admin space, searching on the [main Toolshed](https://toolshed.g2.bx.psu.edu/) for the [MultiQC repository](https://toolshed.g2.bx.psu.edu/view/iuc/multiqc/3bad335ccea9) available under the _visualization_, _statistics_ and _Fastq Manipulation_ sections.

## FreeBSD

If you're using FreeBSD you can install MultiQC via the FreeBSD ports system:

```bash
pkg install py36-multiqc
```

_(or `py27-multiqc`, `py37-multiqc`, or any other currently mainstream python version)._

This will install a prebuilt binary using only highly-portable
optimizations, much like `apt`, `yum`, etc.

FreeBSD ports can also be built and installed from source:

```bash
cd /usr/ports/biology/py-multiqc
make install
```

To report issues with a FreeBSD port, please submit a PR on the
[FreeBSD bug reports page](https://www.freebsd.org/support/bugreports.html).
For more information, visit [https://www.freebsd.org/ports/](https://www.freebsd.org/ports/index.html)

## Environment modules

Many people using MultiQC will be working on a HPC environment.
Every server / cluster is different, and you're probably best off asking
your friendly sysadmin to install MultiQC for you. However, with that
in mind, here are a few general tips for installing MultiQC into an
environment module system:

MultiQC comes in two parts - the `multiqc` python package and the
`multiqc` executable script. The former must be available in `$PYTHONPATH`
and the script must be available on the `$PATH`.

A typical installation procedure with an environment module Python install
might look like this: _(Note that `$PYTHONPATH` must be defined before `pip` installation.)_

```bash
VERSION=0.7
INST=/path/to/software/multiqc/$VERSION
module load python/3.11
mkdir $INST
export PYTHONPATH=$INST/lib/python2.7/site-packages
pip install --install-option="--prefix=$INST" multiqc
```

Once installed, you'll need to create an environment module file.
Again, these vary between systems a lot, but here's an example:

```bash
#%Module1.0#####################################################################
##
## MultiQC
##

set components [ file split [ module-info name ] ]
set version [ lindex $components 1 ]
set modroot /path/to/software/multiqc/$version

proc ModulesHelp { } {
    global version modroot
    puts stderr "\tMultiQC - use MultiQC $version"
    puts stderr "\n\tVersion $version\n"
}
module-whatis   "Loads MultiQC environment."

# load required modules
module load python/3.11

# only one version at a time
conflict multiqc

# Make the directories available
prepend-path    PATH        $modroot/bin
prepend-path	PYTHONPATH	$modroot/lib/python3.11/site-packages
```
