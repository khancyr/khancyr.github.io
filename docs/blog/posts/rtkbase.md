---
title: RTKBase with Partner ArduSimple Kit
date:
  created: 2021-04-28
  updated: 2021-04-28
draft: false
categories: 
    - ArduPilot
authors:
  - khancyr
---

![Ardusimple_logo|690x418, 50%](https://discuss.ardupilot.org/uploads/default/original/3X/1/9/19dde28fb85233f3b8ad13b8c04bff75625e5c40.png) 

Hello friends,

What is better than a GNSS(Global Navigation Satellite System)/GPS (Global Positioning System) ? 2 GNSS ? No, it is an RTK (Real Time Kinematic) GNSS of course !

Indeed, if we simplify, classic GNSS units can achieve between 2 and 10m accuracy, when an RTK unit allows it to reach centimeter accuracy.

But obviously, that is expensive ... You may have noticed that ArduPilot welcomes a new partner that is ArduSimple (https://www.ardusimple.com). They are providing a lot of cost effective RTK ready GNSS boards. And they lend me a [RTK Kit](https://www.ardusimple.com/product/simplertk2b-heading-basic-starter-kit-ip67/). I could have redone their GPS heading tutorial with 2 GNSS (https://www.ardusimple.com/ardupilot-simplertk2bheading-configuration/) but I decided to go another way that is how to make a shareable RTK Base !

I will be speaking about GNSS as that is the generic term for those sensors. The GPS is the USA satellites constellation, whereas GNSS are now able to use multiple constellations like GLONASS, GALILEO, etc.

<!-- more -->
# How does it work ?

As you may know, GNSS works by receiving signals from satellites at known places. By receiving the signal of multiple satellites, we could calculate the current position of the receiver. Therefore, if you take 2 GNSS units close to each other, you should have close to the same position but also the same reception error in amplitude and direction ! So, if we know a good GNSS position with centimeter precision, we could calculate the relative error. If we transmit this calculus to other units, it will be able to correct its positions based on the known error ! That is the basic of RTK GNSS technology.

Therefore, in order to make a RTK GNSS work, you need two GNSS. The first one called the Base, that will be at a fixed location with a good GNSS signal and a second one that is generally called the Rover that will be the GNSS to be used. The RTK principle is quite simple. As the base knows its static position well, it will be able to send the rover the GNSS signal corrections that it learns. An unit that received the correction can apply them to its calculations for better results.

To send the correction from the base to the rover, any datalink can be used : a serial link, WIFI, bluetooth, morse code, etc. We will see that later.

You can find more comprehensive information about RTK technology on ArduSimple page here : https://www.ardusimple.com/rtk-explained/

# That is cool but how to use this ?

The more traditional way to use RTK is to bring your two GNSS units on field, set up your base station and wait for it to have a good position. The main issue is that it can be quite long as the more you wait, the better is your precision. So that isn't suitable for fast deployment like most robotics missions.

So what can be done ? RTK corrections are valid around 35km from the base station. So we could install our base station at some known point, like home, and use the internet to get the corrections !

Fortunately, it already exists and it is called NTRIP Caster. You can even find some service online to provide you corrections without the need to set up your base station. Obviously, in most cases this isn't free and can be expensive.

![Taken from ArduSimple website|602x335](https://lh6.googleusercontent.com/BbXUNKMWD-bMdWL2D5PZmDB_eeGonTtB0WNE-WkE-g2grgMgZq4_xvO65sjEwZKG6ijMqoTmcLzGssYR1ECFYmTZlZMB5hyY3YnGZbj_GIc1MIuq7n42YzGG23VCLi8C3dCtwbG7)

*Taken from ArduSimple website*

What I will show you today is how to set up your own RTK base using an ArduSimple kit and a SBC like a Raspberry Pi.

We got lucky in France as we got a free and collaboratif project called Centipede (https://centipede.fr/) that is an NTRIP Caster free to use and contribute ! Moreover, they provide a good piece of software and documentation on how to setup an NTRIP caster too. Using Centipede allows you to access freely the other bases of the network and profit from RTK corrections, but, as it is french, most bases are in France only. The setup is quite simple. For RPI boards, they provide directly a IMG to burn on SD card to have it working. For other SBC, you can refer to the instructions from the base project [RTKBase](https://github.com/Stefal/rtkbase). That is what we will be using.

Unfortunately, there are some limitations in this software. It is based on a Debian-like distribution and assumes you got nothing else running on your board.

In my case, that was an issue as my boards are using [BalenaOS](https://www.balena.io/) and thus need applications to be into Docker containers. Then I have modified RTKBase to be more Docker friendly and enable it for broader deployment !

RTKBase is quite an interesting software as it allows you to have a nice web interface to manage your GNSS unit to join an NTRIP network, enable a RTK correction streamer, and have some logs. For IoT, it is also useful as it sets up the GNSS as a time reference for the network ! This is really interesting in context where you don't have internet access as your board will be able to send other devices on the network an accurate time clock to prevent all dates to be in 1970 or in 2000.

# What needs to be modified ?

The main issue with RTKBase, besides the install script, is that it depends a lot on systemd components, and those don't play nice with Docker. So in order to have something equivalent, I have rewrite RTKBase to use Supervisord, which is a small python utility that does exactly what its name says : it is an application supervisor. So it is close to systemd for application management.

I had to make some modifications on the GNSS detection script too. This script tries to detect GNSS on serial port and if that is an Ublox unit it will try to configure it. Unfortunately, nothing is direct on Docker so it got some modification to be able to detect correctly the GNSS on the serial port.

My changes are in my own github repository ([https://github.com/khancyr/rtkbase](https://github.com/khancyr/rtkbase) in branch balena), I will try to get them back into RTKBase main repo but this will take some time as I will need my change to coexist with the original systemd implementation.

If you don’t need to use Docker, simply use the normal RTKBase software.

# What can be seen ?

If you are using BalenaOS like me, you can access your RTKBase instance from the internet through BalenaCloud directly without having to set up a complex configuration. On the main page, you can see the status of your GNSS unit.

![|602x460](https://lh5.googleusercontent.com/SKxkGStfNMwVfCR5j4iZ_fFuQDWnwf_MPIfWVMlsAhesOjQE59f0ai_ntfjso2OmP4TYr4VuooLmvSV9-Vinsjw9xY_T4e68doCg_xihFI8UKw2K9NQ0W6prBH7NaCtSQW6aHOuQ)

The settings page allows you to configure external connections. In my case, I didn't use the connection to Centipede, as my GNSS is on my balcony in a big city and the signal coverage isn't that great to be shared with others. It provides an example to connect to Centipede by default, but it is quite generic so you could join another NTRIP Caster.

Instead, I use the TCP connection to send my correction to my GCS (Ground Control Station) that will then send them back to my autonomous rover. An ideal route would have been to have a UDP/UDP multicast feature on the base for easier access and management over 4G or WIFI. And a direct connection on the rover (I need 4G dongle on my rover). Those may come in future improvements.

![|602x473](https://lh6.googleusercontent.com/haKHnJm-1GP2_oxdnkmWHtVbMaUA4B4SLBN8_VpJV9K3DU_qfrMn3N7fplufrFz9nGVSAcP0CKjlTSiV6t3UCo4_PltBk0TsEl2fQpImy0UMloTGZYT9zSu4MsX269VBNR-NVdwo)

In my case with Balena, another issue is that BalenaCloud don't allow direct TCP connection, so you need either to setup a VPN or setup a tcp-to-http proxy to get the correction. This should be solved in the future as the Balena team said they are working on improving their cloud interface. I have then used [Zerotier](https://zerotier.com), which is a free VPN service, to get access to the board remotely and access to RTK.

Here is a picture from Supervisord server access from the VPN that shows what is running on my board.

![|602x259](https://lh5.googleusercontent.com/d-1UC4Wc7Cgo07hF7JfIlw5I18u2yPXkWO8Sodh2pyAHw-xuUqZQNDdq-MZ9T5_8x5BqKlFmYStG2UFeG0lp6XTU6ehP76x4y9TmPEM32YR5cuMy6x4Om2vulxJ3m7g-vAJNGxlR)

# Simple architecture

The code architecture is quite simple. It uses 3 Dockers containers:

* RTKBase: The main container. It launches supervisord that manages:
  * GPSd : a Linux GNSS manager application. It is used to get the data from the GNSS unit.
  * Chrony : a Network Time Protocol (NTP) application that will get time from the GNSS unit and can server as time server
  * RTKLib : that is an Open Source Library for RTK GNSS.
  * RTKBase webapp: the webapp to control your Base.
  * Some GNSS configuration helper script : mainly used to configure GNSS from Ublox brand like the one from ArduSimple.
* Traefik : a docker http proxy. It is used to route traffic from BalenaCloud to the webapp. There is currently an issue with Firefox and HTTPS. BalenaCloud serves correctly the webapp on port 80 but as Firefox upgrades to HTTPS by default, the websocket connection to the webapp isn’t upgrading correctly and breaks the control. So use another browser than Firefox …
* Zerotier : That is a free VPN that is quite simple to manage. In my case, it serves to get access to the base through my phone hotspot !

# Results

Here is my RTK Base on my balcony (that is basically the same equipment mentioned in RTKBase wiki). It is composed of :

* a BalenaFin board (a RPI3 compute module). This got BalenOS and the software previously mentioned.
* a POE splitter to get both ethernet internet connexion and power supply from indoors. There is obviously a POE injector indoor to put power into the ethernet cable.
* an ArduSimple F9P GNSS unit with an deported Antenna.

![|523x732](https://lh5.googleusercontent.com/BUeAXBP9gXCY990KvT8fzFJhDngudPY0Vgz62-v9qVSXZ3F-L9X4UynO-3A0BgdDlmE4RYG9J-QnwXF2cmtMv3nsLaA9RGuzT2SbX-iqKbcCRym_HsXs37eUcI4dIS-STtXilYT1)

Here is a view from MissionPlanner, that is an ArduPilot Ground Control Station, that is connected to the base and gets the RTK corrections ! I can now connect whatever vehicle to MissionPlanner to profit from RTK GNSS !

![|602x325](https://lh4.googleusercontent.com/_0nKyCq-ve9BfSOCtSaKYoy92UfK8m8Qynf0BjGd0YybM7mXQkW8nD1W9HzFo36nUzUEVTrOkQlIE3Kq3QugANsQVEYLgnuuTBixgEYdV1PVO5H0dKyMDPPbqHLBma14ePWuAWr7)

Have fun with RTK !
