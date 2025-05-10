---
title: "JUnit Output support in Autotest Framework"
date:
  created: 2023-11-23
  updated: 2023-11-23
draft: false
categories: 
    - ArduPilot
    - ROS
tags:
    - ROS

authors:
  - khancyr
---
![DALL·E 2023-11-23 00.05.54 - A humorous cartoon image of a drone wearing a graduation cap and glasses, sitting at a computer and looking at a screen displaying 'JUnit Test Results|500x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/7/8/7842304720be9c06aa6544e62e198ea8477d9e42_2_500x500.jpeg)


**Hello ArduPilot Enthusiasts,**

We've just merged a new feature for our autotest framework: JUnit output! Here's a brief summary of what this entails. Seasoned developers will probably know all this, but it should be new knowledge for non-developers.

# The autotest framework.

This is the software suite we use each time somebody makes modifications to the ArduPilot codebase. It has the huge task of running tests automatically. ArduPilot’s AutoTest suite allows for the creation of repeatable tests which help prevent regressions in ArduPilot’s behavior. It is based on ArduPilot’s[ SITL](https://ardupilot.org/dev/docs/using-sitl-for-ardupilot-testing.html#using-sitl-for-ardupilot-testing) architecture - a fully-software-based solution.

## What kind of tests ?

We do a lot of tests! Some check code syntax and common writing errors. But we also do unit tests and functional tests.

Unit tests focus on the smallest parts of code—individual functions or methods. They are designed to ensure that each unit of the software performs as designed. These cover most of our math functions.

Functional tests are concerned with the overall operation of the system. They test larger sections of the application, often entire features. They are performed in an environment that closely mirrors production, so with SITL in our case ! There are close to 1000 of these, trying to test most of ArduPilot's features.


![|500x500](https://lh7-us.googleusercontent.com/ES6Qa8lZO2qE3P4CAhUXRuigY2W5ZP7efSJpVoR-rputlhhVTdiEfsGehF2SIzkGoZ9FrhctSZm3ORVb_QuTnZsqvK8xWLIii2kCJ6YZU2rPu4K7mW5cpCazRbSVYzzKdiT2jRfrkqtEL_fvJBXOeBA)

## How much do we test ?

We've got code coverage support. It's a metric used to gauge the effectiveness of tests by determining which areas of the codebase have been tested and which have not.

On 22/11/2023, here are our coverage statistics (https://coveralls.io/builds/64044563):

**Global** : 57.646% Lines

* AntennaTracker : 68.03%
* ArduCopter : 66.01%
* ArduPlane: 71.2%
* ArduSub: 46.38%
* Rover: 61.08
* AP_Periph: 54.55%
* libraries: 56.33%

We can compare with one years ago (https://coveralls.io/builds/54387841):

**Global** : 56.114% Lines

* AntennaTracker :67.05%
* ArduCopter : 64.63%
* ArduPlane: 67.16%
* ArduSub: 33.96%
* Rover: 61.97
* AP_Periph: 58.59%
* libraries: 55.05%

We can see that we are slowly raising our coverage rates. Ideally, we should aim for 100% coverage, but this is challenging, especially in a FOSS project like ArduPilot. Writing tests is quite lengthy and sometimes difficult, which can delay contributions to the project.

## How to use Autotest ?

Well, we've got some documentation about this, so let’s read the wiki! It isn’t hard:  [https://ardupilot.org/dev/docs/the-ardupilot-autotest-framework.html](https://ardupilot.org/dev/docs/the-ardupilot-autotest-framework.html) !

Don’t forget, if you implement a feature and have a test for it, or better yet, you write a test for it, you can run tests individually: [running-a-specific-sub-test](https://ardupilot.org/dev/docs/the-ardupilot-autotest-framework.html#running-a-specific-sub-test)

That saves a lot of time by testing only what you need.

Plus, our shell completion system makes test execution a breeze – get ready to make the most of your Tab key!


## Junit ?

The JUnit format is a widely-used standard for reporting the results of software tests. It's not a formal standard, but rather a convention that has become a de facto standard due to the widespread adoption of JUnit and similar testing frameworks in the Java ecosystem. Test results in JUnit are typically outputted in an XML format. This XML format is structured and easy to parse, making it compatible with a variety of tools and systems for further processing, like continuous integration servers.

Previously, running the test would provide a small summary at the end in your terminal with information like this:

`>>>> PASSED STEP: test.CopterTests1a at Thu Nov 23 00:01:26 2023`

With the JUnit output, you get an XML file in addition that looks like this:

```
<?xml version="1.0" encoding="utf-8"?>

<testsuites>

<testsuite name="Autotest ArduCopter test.CopterTests1a" timestamp="2023-11-16T14:43:23" tests="27" errors="0" failures="0" skipped="0" time="233.136">

<testcase name="PIDTuning" classname="ArduCopter" time="5.101137638092041"/>

<testcase name="ArmFeatures" classname="ArduCopter" time="20.81020736694336"/>

<testcase name="SetHome" classname="ArduCopter" time="11.95091986656189"/>

<testcase name="ConfigErrorLoop" classname="ArduCopter" time="2.7126312255859375"/>

<testcase name="CPUFailsafe" classname="ArduCopter" time="5.354845762252808"/>

<testcase name="Parameters" classname="ArduCopter" time="9.803567171096802"/>

<testcase name="LoggerDocumentation" classname="ArduCopter" time="3.151670217514038"/>

<testcase name="Logging" classname="ArduCopter" time="13.548582315444946"/>

<testcase name="GetCapabilities" classname="ArduCopter" time="0.07680463790893555"/>

<testcase name="InitialMode" classname="ArduCopter" time="2.385150909423828"/>

<testcase name="NavDelayTakeoffAbsTime" classname="ArduCopter" time="7.005534887313843"/>

<testcase name="NavDelayAbsTime" classname="ArduCopter" time="5.902580738067627"/>

<testcase name="NavDelay" classname="ArduCopter" time="5.835424184799194"/>

<testcase name="GuidedSubModeChange" classname="ArduCopter" time="7.143741607666016"/>

<testcase name="MAV_CMD_CONDITION_YAW" classname="ArduCopter" time="14.092246770858765"/>

<testcase name="LoiterToAlt" classname="ArduCopter" time="7.683399677276611"/>

<testcase name="PayloadPlaceMission" classname="ArduCopter" time="9.601360082626343"/>

<testcase name="PrecisionLoiterCompanion" classname="ArduCopter" time="7.640118598937988"/>

<testcase name="Landing" classname="ArduCopter" time="6.856537818908691"/>

<testcase name="PrecisionLanding" classname="ArduCopter" time="13.348423719406128"/>

<testcase name="SetModesViaModeSwitch" classname="ArduCopter" time="0.9609272480010986"/>

<testcase name="SetModesViaAuxSwitch" classname="ArduCopter" time="0.33498406410217285"/>

<testcase name="AuxSwitchOptions" classname="ArduCopter" time="0.5513660907745361"/>

<testcase name="AuxFunctionsInMission" classname="ArduCopter" time="3.2609589099884033"/>

<testcase name="AutoTune" classname="ArduCopter" time="23.587228536605835"/>

<testcase name="AutoTuneYawD" classname="ArduCopter" time="40.96126747131348"/>

<testcase name="NoRCOnBootPreArmFailure" classname="ArduCopter" time="3.4743480682373047"/>

<properties>

<property name="Firmware Version Major" value="4"/>

<property name="Firmware Version Minor" value="5"/>

<property name="Firmware Version Rev" value="0"/>

<property name="Firmware hash" value="d3d0f91c"/>

<property name="Git hash" value="d3d0f91c92a8d76a42d6fd7aa5ecb398bb464d6a"/>

<property name="Mavproxy Version Major" value="1"/>

<property name="Mavproxy Version Minor" value="8"/>

<property name="Mavproxy Version Rev" value="67"/>

</properties>

</testsuite>

</testsuites>
```

Another example with an output converted to HTML (smaller to fit in a picture):
![image|512x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/5/3/53251d4b73c6f4405954fae0efef88c3594f7827_2_512x500.png)


We can now integrate this into many CI systems to make useful statistics on our tests and have a better understanding of what is failing and when. This is an important feature for professionals using ArduPilot in their fleets.

Ha, yes ! To output JUnit xml, just add `--junit` on your autotest invocation, like this :
`Tools/autotest/autotest.py build.Copter test.CopterTests2a --junit


# In Conclusion

The introduction of JUnit output marks a significant step forward in enhancing our testing capabilities. By continuously refining our testing practices, we're committed to maintaining the high reliability and performance standards that ArduPilot users expect.

Stay tuned for more updates, and as always, happy testing (and flying) !