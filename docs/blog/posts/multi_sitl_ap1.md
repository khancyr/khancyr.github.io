---
title: "Multi SITL on homecloud"
date:
  created: 2021-03-22
  updated: 2021-03-22
draft: false
categories: 
    - ArduPilot
tags:
    - BalenaOS
    - docker
    - swarm

authors:
  - khancyr
---
![|624x312](https://lh5.googleusercontent.com/5LdlQTs-WrYVbwpc8Bsm66c3CWzP-vCRsrjTiuoRdhuWvsQehuguEGFWbnTiideus5ZD-gLorbUQzGBUYAXeIxWSSKlJkelUD8mdUvO-whGyiUhDCqeLjEt2YMioY5tCW7NGZdGs)

Hello friends,

You may remember that some time ago, I made a post about multi robot simulation with ArduPilot (see https://discuss.ardupilot.org/t/multi-systems-patrol-simulations/35740/1).

This was during my PhD, I didn't have much time nor funds to push my simulation into a cloud system for larger scale simulation.

Well, that is still not the case! But I got a bunch of small boards that do nothing in my apartment, so I made a small homecloud to make more simulations.

In this video https://www.youtube.com/watch?v=SMISOREe1y4&list=PL6sCNLbHuYxZVRAbnLY5Xxkr5f4rQeJCJ , which is quite long, I tried to explain how to do it.

## The demo

This is a demonstration of the simplest group behaviour : follow the leader (or wolf-pack). This demo relies on ArduPilot simulator : SITL (Software In The Loop). The simulation uses the same code as the real vehicle with some simulated sensor. This is quite convenient to work on the code and do testing as the transition to the real vehicle will be easier. ArduPilot supports a large set of behaviour, and lucky me, the follow behaviour already got a library, so it is just a matter of enabling it to use it.

The behaviour is quite simple. We define a leader that will stream its position to the followers. Each follower uses the leader position to calculate its new position.

On ArduPilot, we can choose who the leader is and the position of the follower relatively to the leader. So, in this demo I made a more complex follow the leader behaviour.

My master leader is drone 1. Drone 2 to 4 are following drone 1. Drone 10 is following drone 2. Drone 11 to 13 are following drone 10. Drone 20 is following drone 3, drone 21 to 23 are following drone 20, etc.

![|624x451](https://lh4.googleusercontent.com/T75FjNIjdmwxJj13STCzVcahRKbydtRiW7UmN8lZRIe1nEcvw5Bf-_by0vYR01f1fniWVu-ZhvGcU60HlFXNHeHocUd7zFj7CLOXByQ-yvUzBjCKZz98WWbAFMmfwzizCNxNIgt3)

The result is what we can see on the video, it is not perfect but it is working.

As I am using a simulation, I use a multicast protocol to allow each drone to get the information from the other. That is the equivalent to broadcast mode on a RF-radio.

In order to launch the simulation, I have made a small python script using Pymavlink. Pymavlink is a set of tools to control drones using the MAVLink protocol. That is the main protocol used by open source drone autopilot like ArduPilot. The python script is in charge of putting the drone in Guided mode that is an automatic mode that keeps the drone in position waiting for an order. Then it issues a takeoff when the drone is ready and then setting the Follow mode. Finally, there is a control loop at the end as the follow mode will disengage if the drone to follow goes too far or in case of issue. The control loop is in charge to set it back.

The drone 1 is using the Python script to stay in Guided mode and got some random target position over the CMAC field that is SITL default position (that is in Australia since ArduPilot mains developers are in Australia).

![|624x684](https://lh3.googleusercontent.com/_TykA9MDq9J2yyBXv-5vTUA_xb8UBoBwVNWEwxgtN-yxJ4isfIE75sgbZZ9NtTMUc1xnIgYEQbCdz_GryH-IKCOAqD5rFhmSaNpb5UPwNBFbiPl8gTjSDpcLxejWJ1ec0V226yyo)

Don’t fear my python script, it is quite long but most is useless for this demo. If you look closely you could see that most functions are just copy-paste from the ArduPilot autotest framework (https://ardupilot.org/dev/docs/the-ardupilot-autotest-framework.html).

This script will be added to the Pymavlink repository as an example on how to use Pymavlink for drone programming.

I could have used LUA scripting to achieve the same behavior as ArduPilot support. LUA scripting is a scripting language that can be used directly on drones. I am more at ease with Python and want to demonstrate how to use Pymavlink for simple tasks, so no LUA

## Homecloud

![|624x468](https://lh6.googleusercontent.com/l9bgqsjPPP4CIjhD63xTvV8I47tUsKRLJjYQUwaFwaP1hWz6ZtvBnAuRn-O8PR8uLgOz3G6DfHKfS5P7pXTCkbpQCld1A-ff7yR8Yz8tXTiM0Z9QVgZz7sunrUzkXUfjcIc2sxWj)

My homecloud is composed :

- 1 raspberry pi 3 : that is 4 cores so 4 vehicles

- 1 odroid C1+ : that is 4 cores so 4 vehicles

- 1 balena fin (RPI3) : that is 4 cores so 4 vehicles

- 1 odroid XU4 : that is 4 cores so 4 vehicles + the GUI

That is pretty conservative, most should be able to use 2 vehicles by core. But I prefer to play it safe with my power consumption and heat management ... those are in my living room, so it isn't nice to have some fans at full speed all day long !

## System setup

I am not a cloud engineer, so I don't really know the best technology to use. I could have set up each board with their own OS, installed docker or Kubernetes and some VPN but that is really a pain to manage on the long term ... So, my choice was set to BalenaCloud.
Their system is quite simple to set up, support docker and project versioning with Git. Bonus that the same system on all boards, so I code only once for all boards and use git push to update them ! They are also offering a cloud GUI, VPN and a public web address to serve a simple web application. That's what I needed. As I put my homecloud on the public web, even through Balena VPN, I have put it behind an isolation router that puts my homecloud on a separate network than the rest of my home equipments.![|624x308](https://lh3.googleusercontent.com/9lpPCWBhFl_wsETmjftOnp2zdOeFnNiZVy8w6vfYy6m8tkOgJnlg0CeRlObCuES2zwr75NuzS2h0ZB0-jO4enfwTQxFboEb6oX5q44-zlzo-9ugBSYCXHCE6EMWHEtxa-qgb7sYY)

### Simulation cloud architecture

The system will be quite simple :

- Each system will be loaded with BalenaOS and the same docker-compose script to spawn the vehicles.

I made a launch script to be able to use their website to control the simulation. Basically, it exposes some SITL parameters and an ON/OFF parameters, to be able to stop the vehicle in need.

- I made a clean docker image for SITL, as I need to use Python the image is still around 200Mo. That doesn't seem much, but considering that for just SITL the whole docker image is 6,3Mo... it is big. Lightweight docker image is important for simpler and faster deployment (and saves a lot of power on storage and transfer).

- The GUI : that is a MAVProxy plugin using CesiumJS. MAVProxy is a Python Ground Control Station (GCS) for MAVLink drones. MAVProxy is connected to the multicast port, gets each drone infos and streams them back to the Webbrowser. The web application updates each drone position and displays it.

- How to handle multiple vehicle connexions to the GCS without doing it one by one ? SITL supports a multicast protocol and most GCS are able to use it ! That means that we only need to set up all vehicles to connect on the same multicast port and the GCS will have all vehicles available ! Unfortunately, the multicast protocol isn't handled well by docker so we need to expose the full network stack to make it work. That isn't an issue for my project as all my boards are on the same network but it could be an issue on a real cloud. A simple solution for this issue could be to use a VPN like Zerotier to make a bridge between the different instances.

- How to play with the vehicle from the web ? That is where BalenaCloud is nice, and they give a secure link to your instance. The downside is that only http port 80 is enabled. So I bypass this restriction with a proxy to be able to use multiple ports and redirect the connection on port 80 to the right port on the board.

The Docker files for those that want to replicate : https://github.com/khancyr/balena-docker_swarm_follow