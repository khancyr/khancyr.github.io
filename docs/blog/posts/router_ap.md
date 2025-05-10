---
title: "MAVLink routing with a Router software"
date:
  created: 2022-02-20
  updated: 2022-02-20
draft: false
categories: 
    - ArduPilot
tags:
    - MAVLINK

authors:
  - khancyr
---

# MAVLink routing on Companion Computers.

![|501x391](https://lh6.googleusercontent.com/ykddd0a5aAYnoHp2J_e2iLNFIE3fLwA37uOgU5n9oR5rX5JYtAnWkz3xbIqQhzRBthqywFuC0FOjJOPLGiR6xerfULcxPh-OkwLrVA9gwM-k9L_lk5FD79Niwzk8I7usmyTX96ar)

One of the strengths of ArduPilot is that we rely on the standardized MAVLink protocol for our telemetry. As our drones gain more capabilities, companion computers are often used with many separate pieces of software that need to have access to the flight controller data.

A common approach to do this is to use [MAVProxy](https://github.com/ArduPilot/MAVProxy), the ArduPilot developer’s main Ground Control Station. MAVProxy relies on a module based approach that allows it to have numerous capabilities, can serve as a MAVLink message router between multiple components and implement the routing of MAVLink messages according to [Mavlink routing protocol](https://mavlink.io/en/guide/routing.html).

Routing components with MAVProxy is simple: we need a `--master` endpoint to connect to the autopilot and then we can setup as many output endpoints (`--out`) as we want.

This approach is simple to set up but has some costs. MAVProxy, is a fully featured ground control station and not a sole message router.

Using the default startup options consumes a lot of resources, and defaults to loading numerous modules. You can see those default modules at the MAVProxy prompt, (a prompt that shouldn’t exist for a simple routing feature…), by typing `module list` :

````

STABILIZE> adsb: ADS-B data support
arm: arm/disarm handling
auxopt: auxopt command handling
battery: battery commands
calibration: calibration handling
cmdlong: cmdlong handling
console: GUI console
fence: geo-fence management
ftp: ftp handling
layout: window layout handling
link: link control
log: log transfer
map: map display
misc: misc commands
mode: mode handling
output: output control
param: parameter handling
rally: rally point control
rc: rc command handling
relay: relay handling
signing: signing control
terrain: terrain handling
tuneopt: tuneopt command handling
wp: waypoint handling
````

## How can we make this better?

### Solution 1 :

Clean up MAVProxy!

First, launch MAVProxy as a deamon with `--deamon` that removes the interactive shell. Next, you want just routing, so the only module you need is `link`. This module will allow you to set as many outputs you want on launch.

Secondly, MAVProxy should be configured to stop adjusting telemetry rates... You may have noticed that MAVProxy and all Mavlink GCS, automatically request data from the Autopilot. This comes from a special routine that will regularly set the telemetry stream rates: the frequency the autopilot should send a defined group of messages. This feature is convenient for a GCS but not for your MAVLink router as it will fight against other software on the same link. For example, if you have MAVProxy connected to the Autopilot and a companion computer connected to MAVProxy, if you ask with the companion computer to have the IMU message at 50Hz, after a few seconds the IMU message will stop coming at 50Hz and revert to 4Hz as MAVProxy will have reset it.

Thankfully, MAVProxy has an option to disable this behavior, like most GCS. For MissionPlanner, it is in Planner configuration options.

Your new MAVProxy command line will be something like:

````

mavproxy.py --master=tcp:localhost:5762 --daemon --default-modules "link" --streamrate -1 --out 0.0.0.0:14552 --out 0.0.0.0:14553

````

`--master` for the autopilot link

`--daemon` for not have interactive shell

`--default-modules "link"` to keep only the ability to set outputs.

`--streamrate -1` to disable messages control from MAVProxy

`--out 0.0.0.0:14552` to set new output

Now you have a clean router for your Mavlink messages.

### Solution 2:

Use another software that just do MAVLink routing : [mavlink-router](https://github.com/mavlink-router/mavlink-router)

This is small C-based software that mostly only does MAVLink routing. It can do the routing of Mavlink messages according to MAVLink routing protocol, do messages filtering and sniffing, and lastly it has a logger system.

Compared to MAVProxy it is very lightweight both in size and CPU usage. However, it is optimized for routing and has limited other capabilities.

If you have multiple pieces of software that need access to the autopilot data stream, mavlink-router is one of the most used options.

### Solution 3:

Use some Linux tools. These aren’t dealing with MAVLink specifically, but can still be used to push MAVLink messages through IP networks. Two are well known:

- socat : [https://linux.die.net/man/1/socat](https://linux.die.net/man/1/socat)

- ser2net : [https://linux.die.net/man/8/ser2net](https://linux.die.net/man/8/ser2net)

Both allow you to create a serial connection to the autopilot and link it to an IP network.

### Solution 4:

Use [mavlink-hub.py] (https://github.com/peterbarker/mavlink_hub). This is another packet-distributor based on pymavlink; it has no routing capabilities and simply sends any packets received on any of its links to each of the other links. Lighter-weight than MAVProxy and written in portable Python, and thus heavier on resources than mavlink-router.

### Solution to avoid : MAVROS

MAVROS is a great piece of software but it serves its purpose at linking ONE and only ONE MAVLink system (one autopilot for example) to ROS. It has some routing capabilities to relay messages but considering the heaviness of a ROS deployment, you shouldn’t use it for routing and especially if you don’t need to use other ROS tools. Prefer mavlink-router or even MAVProxy that are much simpler to maintain and efficient for routing.

## MAVLink routing protocol

[The Mavlink routing protocol](https://mavlink.io/en/guide/routing.html) defines how a MAVLink message should be delivered when a target is settled. It can be resumed by the two following pictures.

***With MAVLink routing prototocl***
![|501x391](https://lh6.googleusercontent.com/xqUjID71IMX2OjIf207HO1FRHWtFdoEP6kEPMRb_hW6TFbeUCDLSHvfaaZsc8chivPeNa3BG_rYQNW5hVUA5HP9k7H_JyQiBxCAsOU6Wnj0-QRji1MwLnU6BO7aX2r59K45xfnZ_)

***Without MAVlink routing protocol***
![|501x391](https://lh4.googleusercontent.com/FW28-XXnE2CoPJy_Outem7xz6JD4kLeK_sqGLIGFKwptWTc_PRebfaoyKbdVtqeUiefuhxLiAVaRmV8T1R1e5NWasv1Y7-GsQvMKoz30kguOFhMqcz5WrT3I2AigrCJOznxIq8o1)

Do you know some other solutions ? Let's us know !