---
title: Boost Your Drone’s IQ with Dual Autopilot Systems Using ArduPilot
date:
  created: 2024-08-27
  updated: 2024-08-27
draft: false
categories: 
    - ArduPilot
authors:
  - khancyr
---

![|303x386](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd5VQx_4BkBUHfqYHVZ-wllxshBu0CDIX-QqROv-jaMhKcEifiqjZolcgM7g1s1piQaHgk30bYsTsQT8dqXXMGK5TpF2Q-IQ1WXKcoo5sWO4D8-5jgc4T7kVQnRGNYVYAnJDwCL1-mTwBwwZAaBblwNp3pd?key=qa2EVEB9klsZakdHqZVVDQ)
*Credit XKCD 2347*

Sometimes we just want to fly a drone with an expensive payload, or we need to test the code of the trainee that he/she swore he/she tested it. In the rapidly evolving field of unmanned aerial vehicles (UAVs), ensuring the reliability and safety of operations is a must have. One of the latest advancements aimed at addressing these concerns is the implementation of dual autopilot systems. In this blog post, we will explore how using a double autopilot configuration with ArduPilot enhances the reliability and functionality of UAVs.


![|540x500](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdrRvxJYZIuScBxd5js7laChOPvYz7fWciDSw4dhPjqkQG753jBI0A5jyMMSU46H_BHTcJjuvqiDbSPEBMeczkL_-xX31517UxvWCIaJ9ivrWwAieW9hoSK-wRp5Pou_WqxGGTOPzizlZ5FkbSr5Eq7rW8?key=qa2EVEB9klsZakdHqZVVDQ)

Because yes, we can do it for years since [October 2019](https://github.com/ArduPilot/ardupilot/pull/12441)

<!-- more -->
## The Concept of Dual Autopilot Systems

A dual autopilot system involves integrating two independent autopilot units into a single UAV. This redundancy is crucial for enhancing the reliability and safety of UAV operations. If one autopilot system fails, the other can take over, ensuring continuous and stable flight. This can also be extended to more than 2 Autopilots. But here we will speak about the simplest form of redundancy : 2 Autopilots and manual switching.

---

## Key Advantages of Dual Autopilot Systems

1. Increased Reliability: With two autopilot systems, the probability of a complete system failure is significantly reduced. This setup ensures that the UAV can continue its mission even if one autopilot fails.
2. Improved Safety: Redundancy in critical systems, such as autopilots, enhances overall safety, making it suitable for missions in challenging environments or where crash safety is a concern.

---

## Practical Applications

The use of dual autopilot systems is not limited to simulations; it has practical applications in various fields, including:

* Commercial UAV Operations: Enhancing the safety and reliability of UAVs used in commercial applications such as delivery, inspection, and surveying.

* Search and Rescue Missions: Ensuring continuous operation during critical missions where failure is not an option.

* Research and Development: Providing a robust platform for testing and developing new UAV technologies and capabilities.

## ArduPilot way : Example with ArduCopter

There are indeed multiple ways to make a redundant system. We will speak here about the “simplest” and cheapest way : 2 autopilot connected and a radio or GCS to switch from fcu1 to fcu2.

### Architecture example :

![|514x676](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfJXNrNOGklqSdusAHhMYEKmn4ih2AQz3e2TwbHOia8gutZCK3HTVauoUTm-ahhyo2Jparr3Azjl4hHVCsLKeivkfiZVDgfp9pm5Rn9nJgYPyTPD87flY1uGaQduRTPgD-aMGPVBnKxNIebcRPQgvQFwuY?key=qa2EVEB9klsZakdHqZVVDQ)

Overall, the requirement is just doubling everything on the hardware side. The main difference is in the way the output from the FCUs is managed. As most motors controllers only accept one input, we need a “mixer” that will allow us to select which FCU outputs are used ! You can find some RC selector device, create your own analog switching board (with some mosfets gates) or a small FPGA will do the job.

One can also notice that the diagram puts a MAVLink/CAN link between the two FCUs. In a logic of redundant control, the idea is to have one transmitter per FCUs and then use MAVLink (with SYSID_THISMAV parameter settled) or CAN bus to pass data from one FCU to the FCU, allowing to lost 1 transmitter and still have full access to both FCUs !

From the software side. We got an issue. Only one FCU will be in control, which means the second one’s will start to build up some error in PID, Attitude, positonning, etc. This would be an issue as it would mean that your backup FCU will be in a critical state when you want to switch on it.

On a good autopilot like ArduPilot, that is unlikely to happen as that faulty state will be detected and the autopilot will trigger the emergency state : RTL, LANDING, Trigger Chute, etc.

This is still an issue since you won’t be able to use the second FCU as backup …

Fortunately, there is a solution built in that is : [STANDBY mode](https://github.com/ArduPilot/ardupilot/blob/master/ArduCopter/standby.cpp). 

So it isn’t truly a mode, more a submode that will prevent error to build up and fault check to trigger unnecessary.

To enable it, there are numerous ways : GPIO input, MAVLink, RC Radio, CAN, LUA script, etc.

Of course, the second FCU doesn’t necessarily need to be running ArduPilot as long as it can cope with not being in control until commanded to do it.

## Demo on SITL, when simulation is harder than reality.

ArduPilot SITL is able to use the dual FCUs in simulation, however there is some tricks involved.

Indeed, we don’t have a truly simulated “mixer” for now. So in the demo, I use a LUA script to select which FCU output is used to update the simulation, like a mixer would have done. Lua code is here : https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_Scripting/examples/sitl_standby_sim.lua

### Simulation setup :

Two FCUs :

* Set SCR_ENABLE 1, to enable the scripting LUA engine.

* Place the lua script into the script directory

Use RC 8 to select the main or backup FCU

Use RC_OVERRIDE in broadcast to send to both FCUs as the simulated radio input doesn’t work for 2 FCUs, as I said this simulation is harder than reality.

Either :

Patching MAVProxy to send RC_OVERRIDE to both FCU at same time.
Just edit the following file with your favorite text editor : /usr/local/lib/python3.10/dist-packages/MAVProxy/modules/mavproxy_rc.py

And change the self.target_system to 0. This 0 means broadcast MAVLink packet instead of just sending it to the system which is connected to MAVProxy.

![|602x183](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeUlHZRoko2Acf8PFdme50mn0Zyr_XcGfvM-mhP60v0YgwgFqgi1F1GYoIGT7Le0cTI5rJFA6uVSl4yWabkNu-OWO_9ZvS3W4KPwXS4GVV4p-2r3y3rF3WiC7PRWjnJ-KpPcqfxNgdYOJScIK5562Vh-Z_a?key=qa2EVEB9klsZakdHqZVVDQ)

Or wait for the next pymavlink release that got the patch to correct the MAVProxy multilink issues:

![|602x123](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf4_dgCg5NmpXWGlMSglcfEngflpo5kXgtmtd0Js-Ray2NL7bzsU_l5cna_q2-COghey_Zi8QP4-tiCWzuGyB05dtCIPw0U49J2IqsRU6RotUap-4_FeMlbcbaxUwTZoUNfqUiiSF7AWmNaipfNMFYF25U?key=qa2EVEB9klsZakdHqZVVDQ)

### Cmdline FCU 1:

`Tools/autotest/sim_vehicle.py -v ArduCopter -f + --slave 1 -I0 --sysid 1 --use-dir=FCU1 --add-param-file=$(pwd)/2rm.parm -A "--serial1=tcp:5761 " --console --map --no-rcin`

* So we use sim_vehicle.py to launch SITL and MAVProxy to have GCS. Mavproxy will also handle the RC_OVERRIDE.
* We launch a copter with + frame (aka the default), slave say we expect to have a second simulation instance that connects to this one that will be the master.
* I0 for instance 0, to set the default network port.
* sysid 1 to set the MAVLink System ID, as we will use two FCUs, we need to know who sends what.
* use-dir : to run the simulation from FCU1 directory useful to not mix log from multiple FCUs
* add-param-file to use some non default params. Here I just put the SCR_ENABLE 1, so it enables LUA on launch directly.
* -A "--serial1=tcp:5761 " to set serial 1 as tcp server on port 5761, that will be the link with FCU2
* --console --map, to have MAVProxy console and map
* --no-rcin to say MAVProxy to no set the simulated rc radio input but use RC_OVERRIDE

### Cmdline for FCU2:

`Tools/autotest/sim_vehicle.py -v ArduCopter --model json:0.0.0.0 --slave 0 -I1 --sysid 2 --use-dir=FCU2 --add-param-file=$(pwd)/2rm.parm -A "--serial1=tcpclient:127.0.0.1:5761 " --no-rebuild -m "--console --source-system 252" --no-rcin`

Pretty much the same.

* --model json:0.0.0.0 to say we will use the simulated model from FCU1
* --slave 0 to say we are simulation slave from FCU1
* -I1 this is simulation instance 1, so default ports are all offsetted by 10 to not collide with FCU1
* --sysid 2 : this is MAVLink system ID 2
* -A "--serial1=tcpclient:127.0.0.1:5761 ": connect serial1 to tcp port 5761 that should be FCU1 serial1
* --no-rebuild: don’t rebuild SITL binary, as FCU1 cmdline would have already built it.
* --source-system 252: the MAVProxy instance connected to this FCU will have MAVLink ID 252, so we could see which GCS is sending a message.

### Video

https://www.youtube.com/watch?v=SA9kG-Xf-34

#### Conclusion

The integration of dual autopilot systems using ArduPilot represents a significant advancement in UAV technology. By increasing reliability, improving safety, and enhancing fault tolerance, this configuration is poised to become a standard in both commercial and research applications.
