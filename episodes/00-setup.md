---
title: "Software Setup"
teaching: 10
exercises: 2
---

:::::questions
- How to install RAT-PAC2 on various systems?
:::::

::::::::::::::::::::::::::::::::::::: objectives

- Know how to run ratpac2 via a container.
- Compile ratpac2 via `ratpac-setup`.

::::::::::::::::::::::::::::::::::::::::::::::::

## via Apptainer
Installing ratpac2 via containers is the easiest and most reproducible method of installation. If you are not familiar with containers in general, take a look at [Introduction to Apptainer](https://apptainer.org/docs/user/main/introduction.html).

### Installing Apptainer / Singularity
#### Linux
Apptainer (formerly Singularity) can be easily installed in any linux system through a package manager. In most shared clusters, it should already be installed. 
You can most likely install apptainer via:
```sh
apt install apptainer # ubuntu
dnf install apptainer # fedora or rhel/alma9
pacman -S apptainer # arch linux
```
If you are running into issues, please consult the official [apptainer installation guide](https://apptainer.org/docs/admin/main/installation.html#installation-on-linux).

#### Windows
Windows users can follow the linux instruction after installing Windows Subsystem on Linux (WSL) following [this guide](https://learn.microsoft.com/en-us/windows/wsl/install). We recommend using ubuntu 22.04 as your choice of linux environments, but the choice is yours.

#### MacOS
MacOS is not supported by Apptainer. You can either read along to find the manual install procedures, or try to [get apptainer working with a lightweight virtual machine](https://apptainer.org/docs/admin/main/installation.html#mac).

### Running RAT from a container directly
We provide a container with the latest version of RAT-PAC2 built in. You can download the container via:
```sh
apptainer pull ratpac-two.sif docker://ratpac/ratpac-two:nightly
```
Afterwards, you can enter the container by running:
```sh
apptainer run ratpac-two.sif
```
which will drop you into a shell with rat installed. Or, if you can run rat directly by using:
```sh
apptainer run ratpac-two.sif rat macro_to_run.mac ...
```
::::::::::::::::::::::::::::::::::::: caution

There's some known issues with this way of running rat. Notably, it is very likely that geometry visualizations will not run like this.

::::::::::::::::::::::::::::::::::::::::::::::::
### Compiling RAT inside the container (recommended)
We also provide a container with all the dependencies required to compile RAT-PAC2 installed. This is the recommended way of installing RAT for the purpose of developing rat or poking around the code in general.

First, make sure you clone the code repository:
```sh
git clone git@github.com:rat-pac/ratpac-two.git
```
Pull the following container instead:
```sh
apptainer pull ratpac-two-base.sif docker://ratpac/ratpac-two:latest-base
```
Afterwards, enter the container:
```sh
apptainer run ratpac-two-base.sif
```
By default, apptainer should have mounted your current directory into the container. You can quickly check this by making sure that all the files in this container is still there by running `ls`. 

At this point, you can double check that all the dependencies are indeed included in your current shell. Namely, you can run `which root` and `which geant4-config`. Both of them should point somewhere in `/rapac-setup/local`.

If all goes well, you can then compile RAT-PAC2 following the standard compilation procedure:
```sh
cd ratpac-two
make # add -j $(nproc) if you would like to compile on multiple cores (recommended).
source ratpac.sh
```
You can test that your installation is functional by running:
```sh
rat macros/validation/electron.mac
```
A simulation should be performed, and rat should exit with no error. You should now see a log file as well as `output.root` in your directory. Congratulations, you now have a working version of RATPAC2!

## via ratpac-setup
We also provide a script to facilitate dependency installation and management. You can find it [here](https://github.com/rat-pac/ratpac-setup#).
::::: caution
`ratpac-setup` is written primarily with Ubuntu 22.04 in mind. However, many RAT developers have tried it out in different distros, including fedora and arch linux. MacOS should work as well.
:::::
There are some packages that ratpac-setup assumes to be present in the operating system. You can make sure that they are installed with the following one-liner in ubuntu:
```sh
apt install git curl build-essential vim libx11-dev libxpm-dev libqt5opengl5-dev ssh cmake \
    xserver-xorg-video-intel libxft-dev libxext-dev libxerces-c-dev \
    libxkbcommon-x11-dev libopengl-dev python3 python3-dev python3-numpy \
    libcurl4-gnutls-dev ca-certificates libssl-dev libffi-dev
```
Most of these packages should already be present on a standard linux system. On distros other than ubuntu, you probably need to change some of the package names. Please reach out to us if you run into trouble with this.

You can then install the needed dependencies using `setup.sh`. There's many packages provided. We recommend at least the following:
```sh
./setup.sh --only root geant4 cry tensorflow nlopt hdf5 chroma
```
You can append `-j$(nproc)` to use all available CPUs on your system for compilation. This is highly recommended, especially for root and geant4, since they take quite a while to compile.

After this is complete, you can then access all the installed packages in your shell with `source env.sh`. Then you can follow the standard RAT compilation instructions:
```sh
git clone git@github.com:rat-pac/ratpac-two.git
cd ratpac-two
make # add -j $(nproc) if you would like to compile on multiple cores (recommended).
source ratpac.sh
```
and _voila_!

::::: callout
You can also install ratpac-two itself via `./setup.sh ratpac`, but this is not recommended if you plan on working on the code yourself. 
:::::
::::: caution
A common mishap with this method is that you are running these commands in a shell polluted by other enviornment variables. Be extra careful that you do not have another instance of root or geant4 installed somewhere else on your system that's polluting your environment variables, as this might cause the compilation procedure to misbehave. Double check that there's no reference to external geant4/root installed in your `~/.bashrc`. You can also examine the environment variables currently set in your shell by running `env`. Make sure that there's no variables pointing to binaries / data files elsewhere in your system. 

Whenever a installation breaks, the first thing to try should be to close out of your terminal / ssh session, and try the install again in a clean shell.
:::::
