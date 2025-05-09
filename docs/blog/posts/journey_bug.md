---
title: A Journey to fix a bug
date:
  created: 2023-09-18
  updated: 2023-09-18
draft: false
categories: 
    - ArduPilot
authors:
  - khancyr
---

![|602x357](https://lh3.googleusercontent.com/QQZSnJyuQvPC5q015BE-eEeDom9HoXL1BfWD8NNs30lyJaApy0JxdI6bipFZNBybvK32xSeKZjWjfYK0qt6Z_wO6zp7ZfcXd7RBDmOfF4g0T9Jm7zOKmFMcAibpVOnXvyLUJX_XHM01P8PAvrZijqnQ)


Several years ago, I shared a blog post about creating an [RTK base station](https://discuss.ardupilot.org/t/rtkbase-with-partner-ardusimple-kit/70567) and transmitting data via an NTRIP caster, which worked flawlessly at the time. However, after some time, people encountered an issue with Mission Planner's connection to Centipède, the NTRIP caster in my demo. In this blog post, I will detail my journey to fix this problem and bring back the seamless functionality of my setup.

<!-- more -->
## The investigation

To start, I needed to identify the culprit behind the issue—was it Mission Planner, Centipède, or perhaps both? 
To gather clues, I looked on reports on discuss and noticed reports suggesting that it had worked with previous Mission Planner releases, implying that a change in Mission Planner might be the source of the problem.

Upon exploring Mission Planner's GitHub repository, I located the NTRIP connection code in[ ExtLibs/Comms/CommsNTRIP.cs](https://github.com/ArduPilot/MissionPlanner/blob/e5a2524ad6d9b9de6514cc2cd7fa715f8e773307/ExtLibs/Comms/CommsNTRIP.cs). Examining the commit history, I discovered that a few months after my initial post, the NTRIP connection protocol had transitioned from V1 to V2. To gain a better understanding of these protocols, I researched and found explanations on the differences between NTRIP V1 and V2 formats at[ this link](https://www.use-snip.com/kb/knowledge-base/ntrip-rev1-versus-rev2-formats/).

![|602x573](https://lh6.googleusercontent.com/SCXk6hOJ7XRAu85gzjhZbrrpBLmYKLn4s0UEilHd8fekeSLcy4_aNAlSGOnrC41EFLtjV8JmHEzrZmD8vgJ2bGQ-TGl0gNQYbyKzNsazHPJpY__5PhVtBuBUFFe0ev-z0qu2fpH43O4fIYsGk1S9UK0)

![|602x577](https://lh4.googleusercontent.com/Xkn4CgdJoeOo7pNSxWOHePgcO7Bx4F8EDTTdscMoMmZViN00ZGWi5fYjeK23DRTjqT6YeLR0Ihvep7XiZreKTPtmMpGEUf6qFH2EpwMySXUfCpSNuX4o5AwPdtRHdWnKxsXzA5mBzOgtXrr2PguNgmo)

According to the information I gathered, NTRIP V1 should return:

`ICY 200 OK<cr><lf>`

While NTRIP V2 should return:

`HTTP/1.1 200 OK<cr><lf>`

I then attempted to connect Mission Planner to Centipède (http://caster.centipede.fr:2101/TCY22, the nearest caster to my location) and observed that the caster responded correctly with V2 protocol, but data retrieval remained elusive. ![|602x631](https://lh3.googleusercontent.com/OhPIg9M3MB8Xh7a8-WvEZ0tACd9trp08vWUUlXsvjmP3gWpWjCeMoG-oQxlWBczdSmfCdYNbPiwZMM29WtEOHjxDd07LGGb1lQcq2AiTKor9lN7A-iVnpbyncX_-gbf2ZvrdtnSCDJX7auJFY77e_5g)

This led me to question whether Mission Planner was at fault.

Upon further investigation, I tested the connection with rtk2go, a free NTRIP caster (http://email:none@rtk2go.com:2101/JFSM), and it worked seamlessly. This confirmed that Mission Planner's NTRIP implementation was not the issue. (replace @ in your email with -at- to make it works)

![|602x277](https://lh5.googleusercontent.com/liDrIo1cElvudTkrE4VcfKGepIKwHHjdVcvt0b1vAjN0qLfqSsiDpJZDoxZe41eSKRlzyCK7NcWED4NNG_pgkUNwYi9uicivOMHL_PUDF9CpE61t0y2cFLvj4gH4NmxgZjej3juYsid441Br7PZJbOg)

The conclusion: Centipède was the likely culprit. To validate this hypothesis, I decided to test Centipède with another Ground Control Station (GCS), Mavproxy, which includes an [NTRIP module](https://github.com/ArduPilot/MAVProxy/blob/master/MAVProxy/modules/mavproxy_ntrip.py).

Surprisingly, it worked well with Centipède. However, a closer look at the code (https://github.com/ArduPilot/MAVProxy/blob/master/MAVProxy/modules/lib/ntrip.py#L272) revealed that Mavproxy utilized NTRIP V1 protocol by default.

So V2 protocol doesn’t work but V1 does ?

So, while V1 worked, V2 did not. It appeared that Centipède, which relied on the RTKLib backend (https://www.rtklib.com/), was the root cause. Further examination of the RTKLib codebase revealed an issue (https://github.com/tomojitakasu/RTKLIB/issues/592), indicating that NTRIP V2 was not supported by RTKLib. Consequently, Centipède and possibly other systems relying on RTKLib as their backend were affected.

## How to fix the issue then ?

The ideal solution would be to implement NTRIP V2 support in RTKLib, but this would require C black magic. On the other hand, we can enable back V1 support on Mission Planner even if I am no a C# developper.

## Modifying Mission Planner

Since I wasn't familiar with modifying Mission Planner, I referred to the wiki for guidance (https://ardupilot.org/dev/docs/building-mission-planner.html#building-mission-planner).

Unfortunately, I encountered outdated instructions.

![|602x101](https://lh4.googleusercontent.com/8BDnan9vkQUbr5u5CSFM1GCIpnCPKrI0nsbFLim52B9W-pp3Lkn3Uh3p0uekzEFA9m1veVZPrMIJVOBlLs1_I8Ep9y_JGjPIVQFrD3sXgnldhYIEsPSnXo4OodnwOkNTMjM-itsqn87Nc6Mjx8LG-FY)

Fortunately, the Mission Planner GitHub repository had more up-to-date information in its Readme (https://github.com/ArduPilot/MissionPlanner).

I proceeded to create a simple [pull request (PR)](https://github.com/ArduPilot/MissionPlanner/pull/3172) to add a checkbox in the NTRIP connection settings, allowing users to select NTRIP V1 Protocol instead of V2 . So C# master would be able to make an automatic failover logic. However, as I mentioned earlier, it wasn't straightforward since a caster like Centipède responded with `HTTP/1.1 200 OK<cr><lf>` —the correct answer for V2 connections.

As the wiki instructions were outdated, I took the opportunity to update them to benefit others (https://github.com/ArduPilot/ardupilot_wiki/pull/5418).

Lastly, due to slow building times for the wiki, I submitted another PR (https://github.com/ArduPilot/ardupilot_wiki/pull/5418) to enhance the building process's speed.

## Results :

In summary, my efforts led to the following outcomes:

* A simple checkbox in Mission Planner for selecting NTRIP V1 Protocol.
* Working Centipède NTRIP Caster and any using NTRIP V1.
* Updated wiki instructions for others facing similar challenges.

![|602x328](https://lh5.googleusercontent.com/zqUXxaGyvEMiV-TQS5GL2u5nA7ybauqo3sTtlN-tJbsQ2F9x_dZO2yoWXkA141sEMD8EBMyIwtAQ7VVeshvMiBPzZ-DfNmV6aFoJ8_J2XBhG1J5Yt4bhqZyHFfv17SozZDOzhkWS6mnlUOkvbMNfKgg)

With these changes, I hope to make it easier for developers to address similar issues in the future and maintain the smooth operation of RTK-based systems like mine.
