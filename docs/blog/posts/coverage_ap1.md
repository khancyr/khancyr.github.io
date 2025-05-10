---
title: "ArduPilot Code Coverage"
date:
  created: 2021-06-07
  updated: 2021-06-07
draft: false
categories: 
    - ArduPilot
tags:
    - coverage

authors:
  - khancyr
---
# Code Coverage at ArduPilot

![](https://www.commitstrip.com/wp-content/uploads/2017/02/Strip-Ou-sont-les-tests-unitaires-english650-final.jpg)
*Credit to CommitStrip : https://www.commitstrip.com/en/2017/02/08/where-are-the-tests/?setLocale=1*

You may have seen in the latest weeks that the devs were speaking about Coverage. What is that ?

Coverage or code coverage is a technique to gather statistics on which line and function on the code we actually test !  
You may be aware that at ArduPilot we have [an automated test suite](https://ardupilot.org/dev/docs/the-ardupilot-autotest-framework.html) that runs numerous tests each time someone proposes a contribution. But an important question remains : Does the test suite cover the code change ? That is the whole point of code coverage analysis that gives us lines by lines, and functions per function, which ones are called and which ones aren't. The more your tests cover line of code, the less chance you have to have bugs. That is why certification in Aeronautics or in the car industry is long and costly since reaching 100% coverage is close to mandatory and hard to achieve ... resulting in the unfortunate scandals, these two industries have faced in the past few years.

That is why in the latest weeks, Peter Barker and myself push some effort on getting better support on code coverage. We are now getting a simpler script to run the coverage gathering and we will get automated statistics updated every week.

The most important question that you are now asking is : how much code coverage do we have ?
On 2021-05-20, we were on the whole project at:

| Lines |  52.2 %|
|--|--|
| Functions | 61.9 % |

We can also details the statistics per vehicles :

| Copter |  |
|--|--|
| Lines | 64.1 % |
| Functions | 80.8 % |

| Plane |  |
|--|--|
| Lines | 61.5 % |
| Functions | 80.5 % |

| Rover |  |
|--|--|
| Lines | 57.7 % |
| Functions | 78.7 % |

| Sub |  |
|--|--|
| Lines | 33.9 % |
| Functions | 53.4 % |

That isn't that bad but not the best either. We can also see that our testing is unequal among the vehicles, the Sub being the less tested vehicle.
You can have access to our latest report on our server at https://firmware.ardupilot.org/coverage/

## How do we generate this

To gather the code coverage statistics, we are running all our tests ! That means :
- Unit tests : Those are simple tests on functions to test that one input gives the expected output. We don't have much Unit tests, but most of them are in AP_Math library, to test our maths functions https://github.com/ArduPilot/ardupilot/tree/master/libraries/AP_Math/tests
- Functional tests : Those are autotest. We are running simulations test cases with a fully simulated vehicle and test whatever we want : Mavlink message input, sensor failure, autotune, RC input, etc. We got around 300 autotests running currently and the number is growing.

There is now a script [`run_coverage.py`](https://github.com/ArduPilot/ardupilot/blob/master/Tools/scripts/run_coverage.py) in `Tools/scrips/` that allow you to do the coverage testing. You can use it like that :
- First you need to set up your build configuration and build the right binaries. We need to build the SITL binaries before running the coverage tools ! And those need to be built in `coverage mode`, obviously, and `debug mode`, to minimize the compile optimisation. The invocation is `Tools/scripts/run_coverage.py -i`,  where `-i` stands for init. It will then check that you got the right binaries with the right compilation flags. If that isn't the case, it will build them. And finally, it will initialize the code coverage handling with the binaries you built.

You can now launch as much testing as you want, running SITL or making some corrections on the code. Each time you will launch the tests, the coverage handling will run. For example, you can run the Rangefinder drivers testing with : `Tools/autotest/autotest.py test.Copter.RangeFinderDrivers`

To display the coverage, you need to ask for the statistics
- `Tools/scripts/run_coverage.py -u` will do it for you. At the end of the script will ask you to open the `index.html` that will be in the `reports` directory. This will open the same kind of web page with the coverage statistics than on our server.

To run every tests,
- `Tools/scripts/run_coverage.py -f`, where `-f` is for full, will do the building and run all tests. This is really long : around 2h30min to launch every test. That one current limitation of our autotest suite : we don't do parallel testing yet.

## Why does code coverage matter

Doing code coverage analysis when writing tests is a good exercise to understand the code and check that we are truly testing what we want.
It is something to write tests, but it is better if the tests are right ! During the writing of the code coverage script, we find numerous bugs into the code base. Here are some examples :
- `Autotest.py` wasn't passing all arguments correctly : https://github.com/ArduPilot/ardupilot/pull/17554
- On AP_Rangefinder, we use a virtual function in the base class constructor. This is an issue as the compiler doesn't know yet about the derived class. It led to some driver functions not being called on initialization. Hopefully, those aren't critical. https://github.com/ArduPilot/ardupilot/pull/17660
- More testing for AP_Math to have the library closer to 100% code coverage with only unit tests : https://github.com/ArduPilot/ardupilot/pull/17609 .

## What is next

As you have seen we don't have the best code coverage, we are looking to improve this. You can totally help to make the project better by creating new unit tests or functional tests ! This is a good way to learn about the code and contribute to the project.