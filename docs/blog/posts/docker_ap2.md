---
title: "ArduPilot and Docker Part 2"
date:
  created: 2023-03-23
  updated: 2023-03-23
draft: false
categories: 
    - ArduPilot
tags:
    - docker

authors:
  - khancyr
---

![ap_docker|690x174](https://discuss.ardupilot.org/uploads/default/optimized/3X/9/7/97da9102c35baff2b07d0e4a6311dbb531f7ce09_2_690x174.png)

There are numerous ways to use ArduPilot SITL with Docker, so here are some ideas on how to do it fast and lightweight.



The first and easier way is to use our default [Docker image](https://github.com/ArduPilot/ardupilot/blob/master/Dockerfile) that you can find on the ArduPilot root directory. You can build it following the instructions on the wiki at https://ardupilot.org/dev/docs/building-setup-linux.html#setup-using-docker.



But as you can see, the image is large and long to build as it installs a lot of stuff that we don’t necessarily need.



	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
    
	╰─$ docker build . -t ardupilot_base
    
	[+] Building 141.7s (20/20) FINISHED

 

	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
    
	╰─$ docker images

	REPOSITORY TAG IMAGE ID CREATED SIZE
    
	ardupilot_base latest  62620a6f257d 5 minutes ago 1.61GB



And this is just the environment to build SITL !

We could do a little clean up and remove the support for STM32 that we don’t need for SITL, we can reduce the image size and build time a bit. To do this, just add DO_AP_STM_ENV=0 to the ENV directive before the call of [install-prereqs-ubuntu.sh](https://github.com/ArduPilot/ardupilot/blob/master/Tools/environment_install/install-prereqs-ubuntu.sh).


	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
    
	╰─$ docker images
	ardupilot_base2 latest  78618c4ff729 13 seconds ago 905MB



But how to build SITL now ?

Fortunately, Docker brings us a simple way to do this : Multiple stage build !
That means that we will build a new docker image using a previous image as the basis, and in this case the image we just built.
So using ardupilot_base as the basis we can now clone ArduPilot at the revision/release we want. In the following part of my writing I will keep using the master branch.

A git clone should do it, followed by a git submodule init and update as always.

	# syntax=docker/dockerfile:1
	FROM ardupilot_base AS builder
    
	WORKDIR /ardupilot
    
	RUN git clone https://github.com/ardupilot/ardupilot.git src --recursive

Here is the result:

	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
    
	╰─$ docker images
	ardupilot_base_git latest  3ebb0082c065 14 seconds ago 1.98GB

Hum hum, that is quite large again. Let’s shrink this. We just want to build a single SITL binary, so we don’t need the full ArduPilot git history, nor all submodules.
After a bit of cleaning and a faster building time as we will get less things from the net :
# syntax=docker/dockerfile:1
FROM ardupilot_base AS builder

	WORKDIR /ardupilot
    
	RUN git clone https://github.com/ardupilot/ardupilot.git --depth 1 --no-single-branch src \
    	&& cd src \
    	&& git submodule update --init --recursive --depth 1 modules/mavlink \
    	&& git submodule update --init --recursive --depth 1 modules/uavcan \
    	&& git submodule update --init --recursive --depth 1 modules/DroneCAN \
    	&& git submodule update --init --recursive --depth 1 modules/waf

Here is the result:

	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
	╰─$ docker images
  	 ardupilot_base_git2 latest  1627b855ce66 9 seconds ago  1.31GB

From this we can build the binaries we want into Docker. So the final image with the binary we want , in my case arducopter only, is there and we can run SITL, either directly from the binary or with sim_vehicle.py. Remember : SITL != sim_vehicle.py. SITL is ArduPilot simulation, sim_vehcile.py is a convenient python launcher for SITL !

But that is still a large image :

	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
	╰─$ docker images
  	 ardupilot_base_git3 latest  24989f5f4cb1 About a minute ago 1.4GB

For a simple binary 4,8Mb ! Let’s solve this by adding another stage to the docker build.
We are lucky, SITL itself just needs the libstdc++ library to work ! But indeed that means no sim_vehicle.py support ... We can solve this later ([Dockerfile here](https://github.com/khancyr/ardupilot_dockersitl_example/blob/main/base/Dockerfile))

	╭─khancyr@pop-os ~/Workspace/ardupilot ‹master●›
	╰─$ docker images
	ardupilot_base_ap latest  95f0278ba624 4 seconds ago  77.8MB

So that's what we can get ! Just 77.8MB for a working binary, that is the most portable and usable. sim_vehicle.py is useful but for fixed launches, calling the SITL binary is quite simple, so we don’t necessarily need it, and that reduces the image complexity as we don’t need to install Python, just to launch SITL !



## Optimize the docker base image

Instead of using the docker image we get on ArduPilot, we could build a new builder image that is much simpler.
I will detail some approaches on how to do this by reducing the dependencies to the strict minimum. It uses the same multi-stage technique as before but with a different base image. Therefore, I will detail the builder image and the final image with the SITL binary.

### Ubuntu builder [Dockerfile here](https://github.com/khancyr/ardupilot_dockersitl_example/blob/main/ubuntu/Dockerfile)

The builder is :

	ardupilot_ubuntu latest  d979536e6336 5 minutes ago 893MB

That is much better than the previous base builder.
The result with just the binary :

	ardupilot_ubuntu latest  9775cda55321 19 seconds ago 82.8MB


That is a bit larger than the previous build as this was done with Ubuntu22.04 and not Ubuntu20.04 anymore.



### Docker python image [Dockerfile here](https://github.com/khancyr/ardupilot_dockersitl_example/blob/main/python/Dockerfile)


If you still want full Python support, you can still use the default Python Docker image. Please don’t try Python2, it is getting harder and harder to support, so just use Python3. Notice, this is Debian and not Ubuntu, so if you want to install some other packages, the name and version can change !

The builder is :

	ardupilot_python latest  1d9feacc9510 7 minutes ago 902MB

Still better than the default builder.

	ardupilot_python latest  fc61d9da0d83 About a minute ago 132MB

A bit larger than the Ubuntu version, but you got full Python support and can use sim_vehicle.py quickly.



### Docker Alpine [Dockerfile here](https://github.com/khancyr/ardupilot_dockersitl_example/blob/main/alpine/Dockerfile)


So Alpine Linux is a specialized distribution for Docker Containers, and we can use it for SITL ! There are a few differences with running a Debian based distro as Ubuntu, first one is that we are using musl libc (https://musl.libc.org/) and not glibc. What does it mean ? For the end users, probably nothing on usage, for the developers, a bit of adaptation. But as always, ArduPilot is already 100% musl compatible ! So no issues with using it.

The second difference is that it is annoying to install Python on alpine … So I don’t use it on Alpine !
![7egboo|690x388](https://discuss.ardupilot.org/uploads/default/original/3X/e/e/ee095d34dc3e4cbb384e8d4c9ead19fc7cc37f62.jpeg)



The build is :

	ardupilot_alpine latest  ce3b0c467592 9 minutes ago 688MB

The result is :

	ardupilot_alpine latest  59bd11308a3d 59 seconds ago 12.6MB

That is probably the slimmer we can do, perfect for cloud !

# How to run it ?

If you use my Dockerfile, there is a small script to store some default parameters and pass parameters from docker cmdline.
A simple run will be :

	docker run -it --network host ardupilot_alpine

Then you can connect any GCS on port 5760, example with Mavproxy :

	mavproxy.py --master=tcp:127.0.0.1:5760 --console --map

And you are done !

You can also directly call the SITL binary, use sim_vehicle.py, etc.

That is all for now, there are surely numerous other ways to do this and to bring more improvement, but that is a good start ! Next step, use all this to deploy a swarm !