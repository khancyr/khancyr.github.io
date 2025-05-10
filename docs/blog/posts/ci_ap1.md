---
title: "CI at ArduPilot"
date:
  created: 2021-03-04
  updated: 2021-03-04
draft: false
categories: 
    - ArduPilot
tags:
    - CI

authors:
  - khancyr
---

![|624x385](https://lh3.googleusercontent.com/5m6gmYH7FeWWppRJ6rmT2_2DYIdLXVUpniGeVXZhZIGB85oVCCq3GNa4qaI1PDL56SiH4ZDi4XlcFtVDWJSIRorfQ1gu3Xsigy0Nw4HuQzPPsaR64IfGoPWzk3VmqTy-GziyWXzN)

At ArduPilot, we are working hard to improve our contribution flow and testing. You could have seen those last years that our Continuous Integration system was greatly improved. We are now testing building against 19 boards for 4 vehicle types (multicopters, submarines, ground and sea vehicles, plane types) in addition to the simulation testing with around 300 functionals tests and unittest !

Those allowed us to bring more change into ArduPilot with greater confidence and allowed us to track down bugs before they reach the codebase.

Nevertheless, working on a large audience open source project can be hard as the code is changing fast and the Pull Requests (code contributions) are often quickly outdated. This brings up two questions : can we still rebase our work on top of the main branch ? How much flash memory the new change will consume ?

ArduPilot is well optimised for microcontroller boards like STM32 brand, but we need to be careful on our flash requirement as we support boards with a 1MB limit.

That is why we just made a slight addition to our CI system. We now have a new rule that will try to do the rebase against the main branch for a representative set of boards : one 1MB limited, one STM32F4 based one's, one STM32H7 based one's. And we also do a binary size comparison. This allows us to directly get access to the change in flash consumption on the CI system.

![|624x311](https://lh6.googleusercontent.com/6zUrR2vlUNGQgOVZGX757rXtW6Eysz2sIRMjMNkORWAtsAYAyMAI9_d6a4HCnko4FsDymHmX7c5bKXpllqeqz9guPGGoNUd0Xhtzs61EvYXuE-X1Rhma3ri5r7JF80IgvmG4m7_5)

There is still more work to improve our CI and workflow system, if you want to help us, you are welcome to !