---
title: "Ubuntu 20.04 support on ArduPilot"
date:
  created: 2020-05-01
  updated: 2020-05-01
draft: false
categories: 
    - ArduPilot
tags:
    - linux

authors:
  - khancyr
---

![font-ecran-officiel-ubuntu-20-04|690x388](https://discuss.ardupilot.org/uploads/default/optimized/3X/6/0/60b64acc14413796e47d7ba09f5b1161668fff49_2_690x388.png)

Hello guys,

A small news to announce you that we officially support Ubuntu 20.04 !

### Ubuntu 20.04 support

Our installation script ([install-prereqs-ubuntu.sh](https://github.com/ArduPilot/ardupilot/blob/master/Tools/environment_install/install-prereqs-ubuntu.sh)) has been heavily clean up and updated for this release without dropping support on old Ubuntu versions. You can also see that more comments and versatility have been added to be able to understand what it does and setup only the things you need. The default behavior isn't changed and will install everything. We also fix some minor configuration issues, and you should see some compilation speed up for STM target like CubeBlack or Pixhawk1 since the ccache configuration has been corrected (see my previous blog post for brief explanations https://discuss.ardupilot.org/t/speeding-up-compilation/52551)

This version won't bring anything new for ArduPilot. But you must be aware that Python2 was dropped as default python interpreter. Thus, Python3 is the default on 20.04. Hopefully, most of our tools are working on Python3, so you won't notice it.

### Docker improvements

On other news, we have updated the docker image for development we provide for faster and simpler development.
The image has been update to allow you to build all the firmware from Docker and use SITL. By default the graphic environment isn't installed to reduce the docker container size to the bare minimal (or close to).  
You can find the instruction on how to use the docker image here : https://ardupilot.org/dev/docs/building-setup-linux.html#id1 . If you want to use only SITL for cloud stuff or swarming with docker, please use the multi-stage image building. SITL binaries only have libc as dependency allowing you to have a minimal sized container for SITL testing ! I will show you some examples next week.

The docker workflow works like that :
- You build a base image on Ubuntu 18.04 (we will migrating when 20.04 will be fully tested) with all our development tools installed. You can see in the [Dockerfile](https://github.com/ArduPilot/ardupilot/blob/master/Dockerfile) some new environment variables :

  SKIP_AP_EXT_ENV=1 SKIP_AP_GRAPHIC_ENV=1 SKIP_AP_COV_ENV=1 SKIP_AP_GIT_CHECK=1

  Those are passed to our [install-prereqs-ubuntu.sh](https://github.com/ArduPilot/ardupilot/blob/master/Tools/environment_install/install-prereqs-ubuntu.sh) to skip installation of some packages like GUI stuff and coverage. You can still modify the variable to install them.

- You launch the image and mount the ArduPilot git directory in the container. This allows you to have a clean image and share the same directory between your computer and the container. It also allows to be  able to get back the built binary out of the docker  container as you won't be able to flash your board directly from docker  (I may be wrong, if a docker master wants you show use how to do it, please comments !).
- From the docker container, you can use all waf commands and tools like sim_vehicle.py. As said previously, you don't have GUI inside this docker image so Mavproxy won't have Map and Console modules for example.

### Completion improvements

Another improvement  is the installation of our Bash completion system on the installation. This  allow  you to use TAB key to have  suggestion of commands on tools like waf and sim_vehicle.py. for those that already have setup their environment, you can still look into the install script how to setup the completion, or check my blog post about completion [here](https://discuss.ardupilot.org/t/shell-completion-for-tools/49432). By default we only install completion for Bash, for Zsh user please check the blog post or [Tools/completion/completion.md](https://github.com/ArduPilot/ardupilot/blob/master/Tools/completion/completion.md) for instructions.