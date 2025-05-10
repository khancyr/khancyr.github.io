---
title: "Plotjuggler Dataflash plotting!"
date:
  created: 2021-07-07
  updated: 2021-07-07
draft: false
categories: 
    - ArduPilot
    - ROS
tags:
    - ROS

authors:
  - khancyr
---

https://youtu.be/ocyd0ikqQfo

Hello guys,

You may be used to MAVExplorer.py or MissionPlanner for [Dataflash analysis](https://ardupilot.org/plane/docs/common-logs.html). I offer you another alternative : [Plotjuggler](https://plotjuggler.io/)

This is a tool that people using ROS may know already, but it is extensible with plugins support. So I made an ArduPilot Dataflash plugin : https://github.com/khancyr/plotjuggler-apbin-plugins

What are the advantages of Plotjuggler :
- it is fast
- it supports livegraphing
- it got 2D graph support
- it got a LUA script math engine
- it got totally unprofessional Splash screen

There is one important issue with the plugin currently. Plotjuggler is expecting lower case file extension for logs, when ArduPilot is using capital `BIN` type file. So you need to rename your logs to lower case `bin`.

Have fun with it !