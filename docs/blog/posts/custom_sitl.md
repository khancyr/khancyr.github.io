---
title: "Customizing SITL Multicopter Simulations: Tailoring Realism to Your Drone"
date:
  created: 2023-08-25
  updated: 2023-08-25
draft: false
categories: 
    - ArduPilot
tags:
    - SITL
authors:
  - khancyr
---

![test_ardupilot|500x500](https://discuss.ardupilot.org/uploads/default/original/3X/8/4/84495150cdfafe95282cb3727fbddee0e5ca091c.jpeg)



# Introduction:

Many of you are likely familiar with the SITL (Software In The Loop) simulator within ArduPilot, which can be found here:[ https://ardupilot.org/dev/docs/sitl-simulator-software-in-the-loop.html](https://ardupilot.org/dev/docs/sitl-simulator-software-in-the-loop.html). This simulator is user-friendly and quick to set up. However, one notable limitation is its physics simulation, which may not be fully comprehensive. While SITL provides a variety of parameters to manipulate your drone and test code features, did you know that you can refine the multicopter physics to align more closely with your drone's characteristics using a simple configuration file? Let's delve into this customization process.

<!-- more -->
# Enhancing Multicopter Physics:

The default multicopter simulation in SITL is a basic quadcopter with predetermined traits, outlined at[ https://github.com/ArduPilot/ardupilot/blob/master/libraries/SITL/SIM_Frame.h#L82](https://github.com/ArduPilot/ardupilot/blob/master/libraries/SITL/SIM_Frame.h#L82).

![|602x1045](https://lh4.googleusercontent.com/XDCaRJIrl5JSWZ0JPFzkxlU_xSIkEpsh0LsFrjBYp6wsJH4UUX8s-bYi2UZ4zt2oKnGRWqtsdGXEPzdfvXBshcLGDZC96jSM5ZDes8nlAeNA_2OVVeQXk4ku9p11Qcqa8gZ7uIsSesvBPDZqDv_Baew)

This means that even if you are operating an octo-quad copter, the performance will mimic that of a quadcopter. Moreover, a funny side effect is adjusting the default battery voltage to a higher value (Try `param set SIM_BATT_VOLTAGE 45` ) can lead to unrealistic behavior due to a mismatch between the simulated motor input voltage and the provided voltage. This discrepancy is intended to reflect real-world consequences; applying 45 volts to 16 volts motors would indeed damage them. So, how can you address this and tailor your own model?

When you run sim_vehicle.py to launch you simulation, you may have noticed this kind of annoying messages :

![|602x157](https://lh3.googleusercontent.com/6m4sQ30CJ5DA6Farn0zyShkKNb7Hdn1zAYLiy_RBRfoR3R-OvUhjPHIBqE0KKvlPmaVGWIu6Sb3dDOzURhHYJnlRp0nO1BvZV1rvBOsLxWv6vUP9sLGrlK2UxH6wBA2VpQTCLnxh7Vb8qUPOjO5Vd08)

These messages point to examples of copter customization, such as the one found at[ https://github.com/ArduPilot/ardupilot/blob/master/Tools/autotest/models/Callisto.json](https://github.com/ArduPilot/ardupilot/blob/master/Tools/autotest/models/Callisto.json). These files define the multicopter physics parameters.

To create your custom drone model, simply craft a new configuration file (like MyAwesomeDrone.json) and place it in the `Tools/autotest/models/` directory.

# Testing the Custom Model:

To test your new configuration, consider the Callisto model. The updated command becomes:

> sim_vehicle.py -v ArduCopter -f Callisto

![|602x452](https://lh6.googleusercontent.com/-n5btNljM1djyvC49FDvGxL16gZJClb56nSlabmTykZYLJzdE3xtqS9V59BM0GuEssgE3jkeqlD0fajqOqfpbkeTYdt67cNd9AO1nBn61xLEQ5cEzaj8TB2gGWsuhAfhyM-JgScnrvuqTTRstV1ydNM)

![|502x345](https://lh4.googleusercontent.com/Z5mKfOkSzyOGO_eBtLzosPKP0yke2AsOPTuMUs35sZQOCzew-4Wb4n6ZZEq5qpveSnbln1cJROiK3F4UfS1sZvoM7V6EfZ1TCrK1NrcSvHps-7GxBnPndXIeQwI2fgBwnQty9wzWpc8Fb0AJm3qg-mI)

This command instructs the simulation to run an octa-quad drone using Callisto physics and parameters, which are stored in the same location as Callisto.json. Keep in mind that modifying the drone's behavior will alter its reactions. Ideally, the behavior should be more akin to your actual drone's performance.

Let’s try with our AwesomeDrone :
Copying Callisto file to MyAwesomeDrone:
> cp Tools/autotest/models/Callisto.json Tools/autotest/models/MyAwesomeDrone.json
> 
> cp Tools/autotest/models/Callisto.param Tools/autotest/models/MyAwesomeDrone.param

Run simulation
> sim_vehicle.py -v ArduCopter -f MyAwesomeDrone

![|602x387](https://lh6.googleusercontent.com/Wj35vENQlkN2kOULVKlZNBTPAR8kTCO-fq_GGXnWqaAqZEe-OybB46I2TqmSdcElIwb_rSlUQhZXebUyFRDl8ZtqpLYlXfHrtNaKQlsFnrlan_4n6X56aAkptlWOMa0wNY_aygo1YBQh60v7uMSwK9A)

![|493x339](https://lh6.googleusercontent.com/_4CaNzdYfdROyMjDLZhE5tCJ92tldIbuOrzNq232PbBMelmMfLKYEVtoLvFoK3QJeQQl8CkaLIOLK8lOuBkdqqRLSv6uctDLu5iwnafZlqHrA7dCnLBFGFQX-AfXUmwT_ReQT5wzEvhJ6qg0gTZX4dY)

Hum that doesn’t work, why ?

# Troubleshooting and Solutions:

Let’s analyze what happened.

The physics .json file will be embedded into the SITL ROMFS (its virtual file system) during the build phase (in pink). During the launch phase (in orange), however, wSITL may not recognize your custom drone, halting the simulation. Remember, SITL runs in the small white windows, not the terminal where you are using sim_vehicle.py, that is just a launcher for SITL !

And here the issue on sim_vehicle.py ! To simplify common vehicle launch, we got a set of default frames and associated parameters. However, our AwesomeDrone is not registered within these defaults, leading to sim_vehicle.py not passing the right launch parameters to SITL.

To rectify this, navigate to : https://github.com/ArduPilot/ardupilot/blob/master/Tools/autotest/pysim/vehicleinfo.py#L185 and replicate Callisto's entries with your drone's name. Of course adjust on need to match your drone.

Let’s just copy Callisto fields and replace with our drone name :

![|602x499](https://lh3.googleusercontent.com/EBRqLonUhzveG1Voo-gELeQMk4a_DPMhoPpCzKcfbdnBGceUXuxrHSzNorJiz9aSO1lPBot2yhWgvXcvtoIFxAZ4Z_KcfG5Gjp5EugM7UuTQT-NjwseSWTSt2QtNrikCDeAXHuTjt1fOcEsw-q412q0)

And run back SITL :

> sim_vehicle.py -v ArduCopter -f MyAwesomeDrone

![|602x243](https://lh6.googleusercontent.com/FxVSYTTnfqI4D7xccMTiUCIV6JeyIf0kzCJeDa3JUCnTWiPWAqVw29iqcefbxQYeStqq89d8EfDGHYNN9c3M0ZtShYlLld8p_tA3wGk7jFKyGyk32l_VxEntfUaOYYRF9Qd8hgeNI7MwGKPshULede8)

![|491x351](https://lh3.googleusercontent.com/Y1ImlPAs1C7TFzJL2LnOS2BnOGkLAomOFvcyudjZdTkvxJZMK7YVSmi2exk5d4RgrptmZt0KrbIOmRuP7tYuDLk9fB0LRTXUNr_rbQf81hypEy95389yDREDZzSpqI_T6Kdv7JlrCiMLnbGZuVLpI4E)

Now, it is working !

# Alternative Launch Method:

An alternative approach that avoids modifying vehicleinfo.py involves passing the model and default parameter files directly to sim_vehicle.py:

> sim_vehicle.py -v ArduCopter --model octa-quad:@ROMFS/models/MyAwesomeDrone.json --add-param-file=Tools/autotest/models/MyAwesomeDrone.param



For those that don’t use sim_vehicle.py as launcher, it is working the same way :

> /home/khancyr/Workspace/ardupilot/build/sitl/bin/arducopter -S --model octa-quad:@ROMFS/models/MyAwesomeDrone.json --speedup 1 --slave 0 --defaults Tools/autotest/default_params/copter.parm,Tools/autotest/models/MyAwesomeDrone.param --sim-address=127.0.0.1 -I0

# Conclusion:

By following these steps, you have gained insight into fine-tuning your multicopter simulations to more accurately reflect your drone's characteristics and learn how to add a new vehicle default frame to sim_vehicle.py !

Those features are quite handy for better simulations but lack documentation and ease of use ! We could improve sim_vehicle.py to automatically catch new frames in the Tools/autotest/models/ directory and add some autocompletion to catch up those models. If you are interested in contributing to these improvements or adding this guide to the wiki, your contributions are warmly welcomed.

Happy simulations !
