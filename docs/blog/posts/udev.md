---
title: "Mastering FCU Connectivity: A Guide to Secure and Efficient Setup on Ubuntu/Debian Systems"
date:
  created: 2021-01-21
  updated: 2021-01-21
draft: false
categories: 
    - ArduPilot
authors:
  - khancyr
---

![|602x344](https://lh7-us.googleusercontent.com/76JbSSvZ5jrXEaUFaNnxKB3N5QXBBm0bumCzMbsV_RIS-259fN8tGBiGNzBiD0kFRokzKSMdlPjD9qvGMMTJD-8Nt_75f5BUlZ7EQ_fSrWFjrswYxOnC6XZWOcUmfQudXBkb5hrmV8HXGiXHzZ6P5LI)

# How to connect FCU

From reading community posts, and even some partners, one recurring issue is how to connect the FCU (Flight Control Unit aka the autopilot) correctly ?

Let’s dive into some examples on how to do it properly on Ubuntu/Debian like computers.

<!-- more -->
## Bad example

First one is the one to ban ! I won’t even mention working as a root user to use a serial port for obvious reasons.

	sudo chmod 777 /dev/ttyACM0

What does this mean : You are enforcing full write, read, executing permission for all users on `/dev/ttyACM0`. So anybody on your system would be able to take control of it. So in terms of security this isn’t great. And you don’t know what is on `ttyACM0`, it could be your FCU or another serial device.

Moreover, it suffers from 2 big drawbacks that is the source of complains : permission is resettled on reboot ! So you will need to run the command again.

And as you don’t know what is on `ttyACM0`, if you reboot your FCU, it will move to `ttyACM1` (or bigger number) and you will need to run the command against these new ports.

Those are two sources of complaint and we will see how to solve them !

## Good enough solution:

First and easier solution : use the proper user group !

	sudo usermod -a -G dialout $USER

The **dialout** group is given permission to access serial ports, and by adding the user to this group, it allows the user to communicate with serial devices without needing to be the superuser. This needs to logout and log in back to take effect.

With this simple command you don’t have to worry about `chmod 777` any ports for your user nor about reboot issue : your user gains permanent access to the serial ports !

That will solve some frustration, but doesn’t address the `ttyACMx` targeting issue ! How to know where the FCU is ?  

## Connecting by id

Let’s consider two cases :
-   your FCU is on an UART port, so its address is fixed.
-   your FCU is on an USB port or goes through a serial adapter, so it suffers from the port name “issue”.

In the first case, you shouldn’t have an issue as the UART port address will always be the same address. For example, on Raspberry Pi, the first UART is always `/dev/ttyAMA0`. Easy case, if the FCU reboot, it will still be at the same address. If the computer is rebooted, the FCU will always be there ! But remember, only one process can control a serial port, this means for example that you cannot use both Mavproxy and Mavros to use `/dev/ttyAMA0` at the same time. Only one will be able to access the port at a time.

For the second case, the easiest way is to benefit from USB descriptors ! When you connect a USB device to a computer, the device sends a series of descriptors to the host to describe its characteristics, capabilities, and other important details.

### How to access those ?

Easiest command : `lsusb`

	╭─khancyr@MAN-LP-0101 ~
	╰─$ lsusb

	Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
	Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	Bus 001 Device 002: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
	Bus 001 Device 003: ID 2dae:1016 Hex/ProfiCNC CubeOrange

So this shows us what is on our USB ports and the description of the devices ! In my case, I got 2 USB hubs, a CubeOrange and a CP210x UART Bridge.

Let’s look more at the Cube. We can see that it is on the first USB bus, and is device 3. It got some IDs `2dae` and `1016`. Those are called Vendor ID (VID) : `2dae` is CubePilot vendor ID. And `1016` is the Product ID (PID) of the CubeOrange !

It would be very convenient if we can use that information to connect ! And we can !

	khancyr@MAN-LP-0101:~ $ ls /dev/serial/by-id/
	usb-Silicon_Labs_CP2104_USB_to_UART_Bridge_Controller_02AB8FCD-if00-port0
	usb-Hex_ProfiCNC_CubeOrange_4A0035000E51313132383631-if00
	usb-Hex_ProfiCNC_CubeOrange_4A0035000E51313132383631-if02

Magic ! The Cube is there ! So we can access it with a new path : `/dev/serial/by-id/usb-Hex_ProfiCNC_CubeOrange_4A0035000E51313132383631-if00`

Instead of `/dev/ttyACMx` . As this is using the USB descriptors to create a path, it doesn’t rely on the OS to assign a port number like for the `ttyACMx`, and you have the guarantee to always find your FCU on this pass. Quite convenient ! One drawback is that the `4A0035000E51313132383631` in the path is an unique id. So you need to know it to write your connection string. If your software support it, you can use `*` in the path to not have to explicitly look of it : `/dev/serial/by-id/usb-Hex_ProfiCNC_CubeOrange_*-if00`

**What about -if00 and -if02 ?**

On the CubeOrange, you have 2 USB connections : `serial0` and `serial6`. So that explains the two id’s you can see.

Of course, all this isn’t only for Cubes ! It works for all devices using USB ports like FCU, camera, serial adapters etc.

## What if we got 2 FCU ?

	╭─khancyr@MAN-LP-0101 ~
	╰─$ lsusb
	Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
	Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	Bus 001 Device 002: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
	Bus 001 Device 003: ID 2dae:1016 Hex/ProfiCNC CubeOrange
	Bus 001 Device 004: ID 2dae:1016 Hex/ProfiCNC CubeOrange

	khancyr@MAN-LP-0101:~ $ ls /dev/serial/by-id/
	usb-Silicon_Labs_CP2104_USB_to_UART_Bridge_Controller_02AB8FCD-if00-port0
	usb-Hex_ProfiCNC_CubeOrange_4A0035000E51313132383631-if00
	usb-Hex_ProfiCNC_CubeOrange_4A0035000E51313132383631-if02
	usb-Hex_ProfiCNC_CubeOrange_4D0030000D51313132383631-if00
	usb-Hex_ProfiCNC_CubeOrange_4D0030000D51313132383631-if02

So we got 2 different IDs using the unique identifier. So it is still working but it doesn’t make identification easier as the IDs are similar and using `*` won’t help anymore.

## UDEV

UDEV is a device manager for the Linux kernel that dynamically manages device nodes in the /dev directory. That is what creates `/dev/ttyACMx` and the `/dev/serial` paths for example. It can set rules defined in configuration files to determine how devices should be handled.

There we need the USB VID and PID to identify correctly the device we want to handle. Udev rules can be written in numerous ways, you can search on the net how to do it. We will see some examples.

First, we need to create a file with a special naming :

	90-ardupilot-udev.rules

And we will place it in `/etc/udev/rules.d/` . You can probably see that this directory already has some other rules. So the file naming should be a `.rules` extension. Then you must know that the files are used in alphabetic order ! Therefore they generally all start with a number to have easier sorting. In our case, we want a low priority rule so 90 is good.

### What to put in it ?

The syntax can be quite complex but easy for simple tasks. Here is our example content :

	# CP210X USB UART
	ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", MODE:="0666", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"

	# FT231XS USB UART
	ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6015", MODE:="0666", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"

	# Prolific Technology, Inc. PL2303 Serial Port
	ATTRS{idVendor}=="067b", ATTRS{idProduct}=="2303", MODE:="0666", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"

	# QinHeng Electronics HL-340 USB-Serial adapter
	ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE:="0666", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"

	# Cube
	ACTION=="add", KERNEL=="ttyACM*", ATTRS{idVendor}=="2dae", ATTRS{idProduct}=="1016", ENV{ID_USB_INTERFACE_NUM}=="00", MODE:="0666", SYMLINK+="CUBE-$env{ID_SERIAL_SHORT}", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1", ENV{MTP_NO_PROBE}="1"

Let’s analyze this. The first 4 rules are for TTL-USB adapters. They set the `0666` permission on the device. To identify the adapters, we are looking at idVendor (VID) and idProduct (PID). Pretty simple, right ?

What about `ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"` ? Those have special meaning for those that want to use a cellular modem (4G/5G) and use the software `ModemManager` . They prevent ModemManager from looking at those ports as cellular modem to prevent any resource conflict. If you have used MAVProxy, you may have seen this warning:  

	WARNING: You should uninstall ModemManager as it conflicts with APM and Pixhawk

Those options prevent issues !

### What about the Cube rules ?

	ACTION=="add", KERNEL=="ttyACM*", ATTRS{idVendor}=="2dae", ATTRS{idProduct}=="1016", ENV{ID_USB_INTERFACE_NUM}=="00", MODE:="0666", SYMLINK+="CUBE-$env{ID_SERIAL_SHORT}", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1", ENV{MTP_NO_PROBE}="1"

This is the same as for serial adapters : Identify a cube is added on usb .

This looks for a Cube when an USB device is added, if it finds one with VID+PID and interface "00", set `0666` permission, prevent ModemManager and MTP conflict. Lastly, it creates a new symlink (a reference to another file/directory) and calls it CUBE-XXXXX, with XXXXXthe CUBE serial number.

`ENV{MTP_NO_PROBE}="1"` This prevents the OS from trying to mount the CUBE as an MTP "Media Transfer Protocol” device. Common use cases for MTP include connecting a smartphone to a computer to transfer photos, videos, and music, or connecting a digital camera to a computer for file transfer.

So if you connect a CUBE, udev will create a new port : /dev/CUBE-1234. If you plug a second one, /dev/CUBE-2345. That is the same behavior that for /dev/ttyACMx , but with easier name and consistant

That is pretty convenient as it makes finding your FCU way simpler ! If you have different types of FCU, you can put the name you like to find them, and not have to look at each ttyACMx device which is the one you need !

Another solution would be to count the number of Cube present and assign them a number, unfortunately udev don't handle this very well, so it some times doesn't work ...

	ACTION=="add", KERNEL=="ttyACM*", ATTRS{idVendor}=="2dae", ATTRS{idProduct}=="1016", ENV{ID_USB_INTERFACE_NUM}=="00", MODE:="0666", PROGRAM="/bin/sh -c 'expr $(ls /dev/CUBE* 2>/dev/null | wc -l) + 1'", SYMLINK+="CUBE%c",  ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1", ENV{MTP_NO_PROBE}="1"

 

### But how to know exactly where the FCU you want is?

The problem with 2 Cubes is that the first to boot will have the name CUBE-1234 and the second CUBE-2345, so you have to know the Cube serial number to know where it is.

We can change our rules for this, to use the USB hub position as rule to set the name. Let’s change our symlink rule for this one :

	ACTION=="add", KERNEL=="ttyACM*", ATTRS{idVendor}=="2dae", ATTRS{idProduct}=="1017", ATTRS{devpath}=="1.1", ENV{ID_USB_INTERFACE_NUM}=="00", MODE:="0666", SYMLINK+="CUBE1", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1", ENV{MTP_NO_PROBE}="1"
	ACTION=="add", KERNEL=="ttyACM*", ATTRS{idVendor}=="2dae", ATTRS{idProduct}=="1017", ATTRS{devpath}=="1.2", ENV{ID_USB_INTERFACE_NUM}=="00", MODE:="0666", SYMLINK+="CUBE2", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1", ENV{MTP_NO_PROBE}="1"

So here we filter with a new rule : devpath that is the usb hub place. This set the Cube in first usb position as CUBE1 and the Cube at second usb position as CUBE2. Quite conveniant, isn't it ?

How to find this devpath ? There are probably numerous ways. One is to use udev tools to get the usb descriptors !

	udevadm info --attribute-walk --name=/dev/serial/by-id/usb-Hex_ProfiCNC_CubeOrange_4A0035000E51313132383631-if00

Udevadm info starts with the device specified by the devpath and then walks up the chain of parent devices. It prints for every device found, all possible attributes in the udev rules key format. Looking at the output, we will found something like :

	udevadm info --attribute-walk --name=/dev/serial/by-id/usb-Hex_ProfiCNC_CubeOrange_4D0030000D5131313238
	3631-if00
	SELinux enabled state cached to: disabled
    
	Udevadm info starts with the device specified by the devpath and then
	walks up the chain of parent devices. It prints for every device
	found, all possible attributes in the udev rules key format.
	A rule to match, can be composed by the attributes of the device
	and the attributes from one single parent device.
    
  	looking at device '/devices/platform/scb/fe9c0000.xhci/usb1/1-1/1-1.3/1-1.3.3/1-1.3.3:1.0/tty/ttyACM4':
    	KERNEL=="ttyACM4"
    	SUBSYSTEM=="tty"
    	DRIVER==""
    	ATTR{power/control}=="auto"
    	ATTR{power/runtime_active_time}=="0"
    	ATTR{power/runtime_status}=="unsupported"
    	ATTR{power/runtime_suspended_time}=="0"
    
  	looking at parent device '/devices/platform/scb/fe9c0000.xhci/usb1/1-1/1-1.3/1-1.3.3/1-1.3.3:1.0':
    	KERNELS=="1-1.3.3:1.0"
    	SUBSYSTEMS=="usb"
    	DRIVERS=="cdc_acm"
    	ATTRS{authorized}=="1"
    	ATTRS{bAlternateSetting}==" 0"
    	ATTRS{bInterfaceClass}=="02"
    	ATTRS{bInterfaceNumber}=="00"
    	ATTRS{bInterfaceProtocol}=="01"
    	ATTRS{bInterfaceSubClass}=="02"
    	ATTRS{bNumEndpoints}=="01"
    	ATTRS{bmCapabilities}=="2"
    	ATTRS{iad_bFirstInterface}=="00"
    	ATTRS{iad_bFunctionClass}=="02"
    	ATTRS{iad_bFunctionProtocol}=="01"
    	ATTRS{iad_bFunctionSubClass}=="02"
    	ATTRS{iad_bInterfaceCount}=="02"
    	ATTRS{supports_autosuspend}=="1"
    
  	looking at parent device '/devices/platform/scb/fe9c0000.xhci/usb1/1-1/1-1.3/1-1.3.3':
    	KERNELS=="1-1.3.3"
    	SUBSYSTEMS=="usb"
    	DRIVERS=="usb"
    	ATTRS{authorized}=="1"
    	ATTRS{avoid_reset_quirk}=="0"
    	ATTRS{bConfigurationValue}=="1"
    	ATTRS{bDeviceClass}=="ef"
    	ATTRS{bDeviceProtocol}=="01"
    	ATTRS{bDeviceSubClass}=="02"
    	ATTRS{bMaxPacketSize0}=="64"
    	ATTRS{bMaxPower}=="100mA"
    	ATTRS{bNumConfigurations}=="1"
    	ATTRS{bNumInterfaces}==" 4"
    	ATTRS{bcdDevice}=="0200"
    	ATTRS{bmAttributes}=="c0"
    	ATTRS{busnum}=="1"
    	ATTRS{configuration}==""
    	ATTRS{devnum}=="21"
    	ATTRS{devpath}=="1.3.3"
    	ATTRS{devspec}=="(null)"
    	ATTRS{idProduct}=="1016"
    	ATTRS{idVendor}=="2dae"
    	ATTRS{ltm_capable}=="no"
    	ATTRS{manufacturer}=="Hex/ProfiCNC"
    	ATTRS{maxchild}=="0"
    	ATTRS{power/active_duration}=="1388444"
    	ATTRS{power/autosuspend}=="2"
    	ATTRS{power/autosuspend_delay_ms}=="2000"
    	ATTRS{power/connected_duration}=="1388440"
    	ATTRS{power/control}=="on"
    	ATTRS{power/level}=="on"
    	ATTRS{power/persist}=="1"
    	ATTRS{power/runtime_active_time}=="1388266"
    	ATTRS{power/runtime_status}=="active"
    	ATTRS{power/runtime_suspended_time}=="0"
    	ATTRS{product}=="CubeOrange"
    	ATTRS{quirks}=="0x0"
    	ATTRS{removable}=="fixed"
    	ATTRS{rx_lanes}=="1"
    	ATTRS{serial}=="4D0030000D51313132383631"
    	ATTRS{speed}=="12"
    	ATTRS{tx_lanes}=="1"
    	ATTRS{urbnum}=="14"
    	ATTRS{version}==" 2.00"


So now you got the exact position on the usb hub, you can use this information to set the right name on each device you got.

## How to trigger udev

After writing your rules, you can trigger udev with :
 

	sudo udevadm control --reload-rules && sudo udevadm trigger -c add

If you didn't use the ACTION=="add", you can remove  `-c add` to trigger the new rules.

## Conclusion: Smarter FCU Connectivity for the Win!

And there you have it, folks – a guide to connecting your FCU in a smart, secure, and hassle-free way. Let's sum it up:
1.  **Ditch the Bad Habits**: Say a big no to `sudo chmod 777`! It's like leaving your front door wide open. Instead, be savvy about permissions and security.
    
2.  **User Group Magic**: Just remember, `dialout` is your new best friend. By adding your user to the `dialout` group, you're setting up a connection that won’t vanish after a reboot. It's like a trusty old friend – always there when you need it.
    
3.  **Descriptor Detective Work**: When it comes to tackling the `ttyACMx` mystery, using USB descriptors is like being a detective. You get to pinpoint your FCU. No more guessing games every time you plug in a device.
    
4.  **Multiple FCUs? No Problem!**: Got a fleet of FCUs? Udev rules are like custom remote id  for each one. You'll never mix up your devices again.
    
5.  **Udev Rules to the Rescue**: Udev rules are not just lines of code; they're your toolkit for a tailor-made setup. With them, you can banish conflicts with pesky programs like ModemManager and keep everything running smoothly.


In short, connecting your FCU shouldn’t feel like rocket science. With these tips and tricks, you'll be navigating the Linux terminal like a pro, and your FCU will be up and running in no time. So go on, give these methods a try, and take control of your FCU connectivity like a boss! Happy flying!



