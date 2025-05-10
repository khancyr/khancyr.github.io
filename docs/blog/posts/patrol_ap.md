---
title: "Multi systems patrol simulations"
date:
  created: 2018-11-30
  updated: 2018-11-30
draft: false
categories: 
    - ArduPilot
    - ROS
tags:
    - ROS
    - swarm

authors:
  - khancyr
---

https://www.youtube.com/watch?v=YLDb-ti9bjU

Hello everyone,

I am often asked about what can be done with ArduPilot or with ROS, and my answers are always : you can do just about anything you&rsquo;d like to do! ArduPilot is not just an autopilot for drones, it is also a well-grown robotic platform ! It is not the same as ROS, in the sense that ROS concentrates more on the high level tasks, whereas ArduPilot focuses on low-level vehicles control ! Both have strengths and drawbacks. And by using both, you will be able to create incredible robots !

During my PhD, I tried to focus my work on multi-robots systems, which are also known as swarms (from an academic perspective, swarms are just a special case of multi-robots systems). ROS by itself can excel for single robot design, but is difficult to use on multi-robots systems and flying robots like multi-rotors. On the other hand, ArduPilot allows to use multi-rotors more simply, and with safe behaviors.

In order to demonstrate the usefulness of multi-robots systems and be able to evaluate the global performance of the system, I set up some identical simulations with ROS and ArduPilot. The goal was to demonstrate the performance of a multi-robot system with various numbers of quadcopters, and rovers on patrolling tasks. I then simulated systems with 1 to 20 quadcopters, and 1 to 20 rovers and system with the same number of quadcopters and rovers. Those were simulated with SITL from ArduPilot, and I will detail at the end the few modifications that I made and that I am pushing into ArduPilot. The patrolling task was done with ROS. Each vehicle got a Mavros instance (that is, a ROS program that makes the bridge between ROS language and MAVLink), a simple python script that handles the patrolling behavior, and a master python script that controls the group. But that was too simple ! So I implemented 3 patrolling behavior, that are quite simple :

- Random patrol (RANDOM). It is a decentralized algorithm. Each robot will go at a random patrolling point

- Reactive patrol (HIGHEST). It is a centralized algorithm. When a robot arrived at his destination, it notifies the master script (like a base) and gets the next patrol point.

- Cyclique patrol (CYCLE). It is a decentralized algorithms. At start, each robot create a patrol cycle.

For the full detail of the simulation models and hypothesis, please refer to my PhD manuscript and/or directly the code !

![image|690x193](https://discuss.ardupilot.org/uploads/default/original/3X/7/f/7f6e8d780f79dd35e9bfdbfc830967083d06852f.png)

The results were those graphs for 1 hour of patrolling on this field, the number represent the patrol points :
![image|402x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/2/1/21fade3ab0c93fea3f89d03ffe6bcb9a8c08dd2d_2_402x500.jpeg)


For copter only:

![image|499x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/7/0/701cccb86de5a799d2d06c287fb84f9cc6ed3245_2_499x500.png)

For rover only:

![image|507x499](https://discuss.ardupilot.org/uploads/default/optimized/3X/7/1/71e947ffc6552f11867b4823da3c434b3fcc2289_2_507x499.png)

For rover and copter :

![image|690x419](https://discuss.ardupilot.org/uploads/default/optimized/3X/8/1/814f5aad12d84236e7ced766feb8e155d949a5ed_2_690x419.png)
![image|690x330](https://discuss.ardupilot.org/uploads/default/optimized/3X/3/e/3e87e7e6f83ac69ac14bcefd96e91558ca8c97ef_2_690x330.png)

All on same graphs :
![image|494x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/b/2/b2f4fc4657d48e3e39bf0d1807bc3461ec1ad886_2_494x500.png)
![image|523x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/2/2/2200753e38b10e9ce57e3b199d39c5b2480ca47c_2_523x500.png)
![image|524x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/7/b/7bfba5a6873b6628fd2a1ef6f17159ccb9d58832_2_524x500.png)

You can see a video here of the patrolling : https://www.youtube.com/watch?v=YLDb-ti9bjU

I won't discuss the results here, but you should be worried by SKYNET ^^ as the group behavior outperform largely the single robot patrolling ! But what it was really interesting, it is that I could make all those simulation on my own laptop : just a i5 + 8go RAM and with simulation accelerated 10 times ! Right ! (If I find the budget to build 20 quads and 20 rovers, I would be able to confirm the simulation with the real platform)

ArduPilot allows to simulate entire fleets of robots with the same code that will be embedded and at a small cost ! That is right, for those kind of behavior, you don't need to use big simulator like Gazebo on big 32 cores servers. Simulate simply with SITL !

I don't say that you should use Gazebo or other well known robotic simulator, but when it comes to simulate only behavior without worrying about collision, SITL with excel !

All these were permitted thanks to OPEN SOURCE and numerous contributions ! PR welcome !

F.A.Q

Where is your code :

Github ardupilot https://github.com/ArduPilot/ardupilot and branch PhD on my fork https://github.com/khancyr/ardupilot/tree/PHD : I am currently getting back those changes into ardupilot master.

Modifications done :
- allow to set sysid at the start of SITL (PR in master)

- disable mavlink routing on a channel. Finally I didn&rsquo;t use this, this was intend to prevent mavlink spam on a broadcast channel for the monitor. I still believe that we need a way to stop mavlink routing explicitly on some channel, as on swarm configuration, there is to much spam.

- clear battery failsafe : I didn&rsquo;t use this, I got another code that I will push to allow to refill the battery and clear the failsafe. On my code, the ROS part is monitoring the battery and decide when to go on base to change the battery.

- disable MSG_POSITION_TARGET_GLOBAL_INT sending, as there is a bug in MAVROS. It has been reported and I need to make a fix on MAVROS.

- Disable some stream and unused features. This won&rsquo;t be push back on master.

- Prevent switching in guided is EKF is not ready for Rover. (Need a PR)

- disarm rover at the end of RTL : I am not sure we want this into master, maybe with a parameter.

- correct battery estimation for rover : PR done.

ROS :

https://github.com/khancyr/patrol

I want to do the same with Gazebo :

If you need only a 3d visualization see https://github.com/khancyr/gazebo_ardu . This can be adapted to any 3D framework you want.

To use the gazebo plugin : https://github.com/khancyr/ardupilot_gazebo . Beware, the simulation of rover isn't simple yet as the ground contacts are a pain and the friction gives a hard time to make a skid steering rover.

Why use ArduPilot over PX4:

PX4 don't support rover and surface vehile like boat and sub. Moreover, the governance of ArduPilot is open whereas PX4 is <strike> money </strike> dronecode driven

I don't understand anything but the colors were nice :

Thanks you.