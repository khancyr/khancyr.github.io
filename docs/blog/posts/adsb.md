---
title: "Learning ADS-B Technology: A DIY Guide"
date:
  created: 2023-09-11
  updated: 2023-09-11
draft: false
categories: 
    - ArduPilot
authors:
  - khancyr
---

![|602x431](https://lh3.googleusercontent.com/Zk2pLEW0Btd7qQwAFOedQYXV_msixr2ffbcoKzl3giBADKByItHMR3JHoKCULQCMX4ggU2EN3eNxfChs9VvM6Or9CG2IEqciN4Lu9WaPlsYcJPh2mu_Xz4kAPEL-pYPPV9r7BSwKSlg8plVi6T4JEw4)

With the application of new regulations all around the world, one noticeable trend is the widespread use of ADS-B (Automatic Dependent Surveillance-Broadcast) detection modules on drones. Prominent companies like DJI have integrated these modules into their fleets, and ArduPilot has been supporting ADS-B integration for years even before DJI, thanks in large part to CubePilot and UAvionics. These small, integrated modules, such as those found in CubeOrange baseboards, are revolutionizing drone safety. In this post, we will delve into the world of ADS-B, exploring how to build your own ADS-B receiver.

<!-- more -->
## What is ADS-B?

ADS-B stands for Automatic Dependent Surveillance-Broadcast, a technology originally designed for aviation purposes. It allows aircraft to autonomously broadcast essential data, including their identity, position, altitude, velocity, and other relevant information, to both ground stations and other aircraft. This information is transmitted using specific radio frequencies.
This is only a resume as you can find good explanations online. 

## Why is ADS-B Useful for Drones?

ADS-B data is broadcasted via radio signals, making it accessible to anyone with the right hardware and knowledge to decode it, and we will see that isn’t much complex nor expensive. This capability provides crucial information, such as the sender's position, heading, and speed, which is invaluable for drones, primarily for avoiding collisions with manned aircraft, a potentially life-threatening scenario. You can find how to use those data into ArduPilot in our dedicated wiki page : https://ardupilot.org/copter/docs/common-ads-b-receiver.html

## How to Access ADS-B Data?

You can access ADS-B data through online services like[ FlightRadar24](https://www.flightradar24.com) or[ FlightAware](https://fr.flightaware.com/). Alternatively, you can set up your ADS-B receiver for a more hands-on experience.

## Building Your Own ADS-B Receiver

Building an ADS-B receiver is not only affordable but also legal. All you need is a computer and a radio receiver. One of the most cost-effective options is to use a USB TV tuner, which can be easily configured to listen to ADS-B signals.

In my case, I repurposed an old Android TV box (approximately €35) and used a FlightAware Pro Stick® Plus (around €50 with an antenna).

The ADS-B community offers various software options, but I opted for Linux and Docker due to my familiarity with them and to reuse the board with other fun projects. I installed Armbian into my Android TV Box, a user-friendly process described [here](https://github.com/ophub/amlogic-s9xxx-armbian) for TV boxes or [here](https://www.armbian.com/) for generic SBC. It was quite nice to reuse some old but functional hardware for this !

For ADS-B decoding, I chose [this guide](https://sdr-enthusiasts.gitbook.io/ads-b/), which is excellent for newcomers because of its straightforward configuration. Follow the guide to install the ultrafeeder and enable as many modules as you want, you can also share your data with some ADS-B aggregators like flightradar24 or flightaware to have a gold account.

![|509x490](https://lh6.googleusercontent.com/dyTNE2tEAooJR4eB5jKPJU5ILMB3OcxQ1o0QmBwta0aKLx1gsNtiSRoenV5Ef8CPkyja4AGfWWCk5EZWi1FEoWXCkF-02TDnJRAxW40Bdk-lI6kJq-Y7VZT2r070gC0r-gsw_5p_F3gH2WTUG9zuzdA)

Depending on your choices, you should have at least the default tar1090 web page (BOARD-IP:8080), displaying the received plane signals. 

![|602x315](https://lh5.googleusercontent.com/Mv5me6oOEVt-HZ6GATMR0WZa7MxWIGSAAOivJrZMnb70tPV-bLnl7tKkJShnjoM94EOaso9obot83i8JcK0U08JXw523_vLJENWusqM8fKx2FSBHcy8D7mUpWufEXFs_nj_MBAAsEsRvt6MhfqSxww0)

While my antenna position may not be ideal (on my roof with no signal filter), it still provides a good tracking range of up to 150km. 

And of course, you can send this to MissionPlanner too.
In MissionPlanner, navigate to the config tab and use the ADS-B tickbox to input your receiver's information. If you're using the ultrafeeder like me, make sure to expose the SBS/Basestation protocol output in your docker-compose.yml file, like this:


```
ports:
  	- 8080:80           	# to expose the web interface
  	- 30003:30003/tcp
```

## Conclusion

With minimal hardware investment, you can explore the fascinating world of aeronautical technology and enhance the safety of your drone flights. However, it's essential to note that not all aircraft emit ADS-B signals, particularly smaller planes and some military aircraft. Therefore, while having an ADS-B system that feeds your Ground Control Station or drone is beneficial, it should never replace your vigilance regarding the surrounding airspace.

In this post, we've only scratched the surface of ADS-B capabilities. The software offered by sdr-enthusiasts also allows you to receive radio communications from planes, ACARS and VDL-M2 messages, and even AIS data related to ships and vessels. Not to mention Planefence, which can notify you when interesting planes, such as military or presidential aircraft, pass over your receiver.


### Family tips: 

If you need a reason to install yet another antenna on your roof, involve your kids in this exciting endeavor. Let them explore ADS-B tracking and challenge them to identify the corresponding planes in the sky. It's a fun and educational activity that can spark their interest in aviation technology.
