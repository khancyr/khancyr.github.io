---
title: Integrated remote controller video sharing (Herelink, Siyi, etc.)
date:
  created: 2025-02-07
  updated: 2025-02-07
draft: false
categories: 
    - ArduPilot
authors:
  - khancyr
---

![|602x385](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeZNGAeglCl1SARXN4FYFti9oTl9FnlN1PVSmsJDSDaWsn4Myt_qDvOwHJNh2MA8FX47ohEqJZSq3ZdClqJyJh_wNufyLOzvVc18mYDIKV6h0Be9_o04VFH6IFYZVNmwm4FR4LU?key=uF1lrlKcv2DMk5fHIqZRlSjX)

In the drone world there are 2 kinds of people : the FPV bros and the other. For the other, sharing video can be important and for this we got a bunch of choices for Integrated Remote Controllers with HD Video support : Herelink, Siyi, etc !

In a wonderful world, everything would work out of the box … but … welcome to the matrix !

So those controllers can share video to the ground according to their documentation : tested on Herelink and Siyi. That is kind of true …

Let’s look at different usages

*(This article isn't to shame or attack any manufacturer but to start some investigation on one issue and find a solution)*

# What is working ?

The only player that has comprehensible documentation on this is CubePilot with the [Herelink](https://docs.cubepilot.org/user-guides/herelink/herelink-user-guides/share-video-stream).

What is explained there is working as intended : Use the remote controller as Wifi/bluetooth/tether hotspot and you can share the video from it to a computer.

I didn’t test, but on Herelink it also looks like you can connect the remote controller on a Wifi router (see the difference) and get your HDMI camera video feed from the ground thanks to a video proxy integrated.

Same thing on Siyi side, with less docs as you need to look at some pdf or youtube videos.

# What isn’t working ?

What is happening when you want to connect your remote controller on a Wifi router and share the video feed from an IP camera (RTSP video) connected to your air unit ? Well, surprised ! You cannot ! Lot of questions on this on the different manufacturers forums and no solutions … So Herelink, Siyi and probably other have the same issue. 
I will continue the post by speaking about the Herelink as that is the one that was lent to me to find a solution.

## What is the issue ?

I got a Siyi camera A2 Mini that has IP 192.168.144.25 and plugged in the Herelink Air Unit (the one that should go on the drone). On the Herelink, I can get the video feed from QGC (use a recent version), Siyi FPV or whatever app that can open rtsp://192.168.144.25:8554/main.264 that is the camera video feed. Great.

But I cannot open this from VLC, for example from my computer. It just failed.
The Herelink has 192.168.4.21 as IP address on my network. I can ping it.
Trying rtsp://192.168.4.21:8554/main.264 doesn’t work, nor is rtsp://192.168.4.21:8554/fpv_stream as stated in the doc. I assume the documentation on Herelink is incomplete there and their video proxy is only working when you are using an HDMI camera on the Air Unit and not an IP camera.

I cannot ping 192.168.144.25, which is a clue that the ip routing or something is wrong.

As the Remote Controllers are Android based, I can use ADB to connect it and try to debug (or use [PingTools ](https://play.google.com/store/apps/details?id=ua.com.streamsoft.pingtools&hl=en-US)application directly on it).

Here is the copy of the network config.

![|602x308](https://lh7-rt.googleusercontent.com/docsz/AD_4nXendJx1vHEK6wmHYftqRwYhDUrfyIR8g5itx_h4r111q_YoyHjCIBx86e1EUDhGhlzmcBNPHeG9H2fHUxwTkHO9KLNH0wqQCOjC0MBlF87_Crexv0DXT3hkhZSgfuGprTfHEa2Q3g?key=uF1lrlKcv2DMk5fHIqZRlSjX)

![|602x132.9558457609234](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcmbs7AcV5NOPxyNsSwIXAJybFgwLdFBpwS8CY6NtMRwOie5IdsbfwVaCj23r7U3aFrZtGkYP176tdZl4XbhPtNLP1hm3hJ610Iq3PyG67_V4tBU2iN7wwxSN4XwuzHZoc0vDaL?key=uF1lrlKcv2DMk5fHIqZRlSjX)

## Understanding the Network

* 192.168.4.21 → wlan0 (connected to amy router)
* 192.168.144.11 → br-vxlan (the bridge in 192.168.144.0/24)

The Herelink is acting as a router between these networks. However, other devices like my computer cannot reach 192.168.144.25 unless routing is properly set up.

## Solution: Enable Forwarding & NAT on Android

Since we don't have root (aka full permissions), we need workarounds.

First, let’s check if 192.168.144.25 responds from Android with ping 192.168.144.25 from adb ! Respond normally. Android sees the device, so the problem is routing. (what is kind of obvious since we can get the video on QGC)

Let’s check IP Forwarding :

![|602x60.14079374437145](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcwCvnZKEhxZ2VpyR-zxXvWYJ5hz-5U8sJkL2LM3MBmXmWzPqTA7b_ggdmB4VxfltUhXtzPqOftiQoV4mutNxr5k1iWVuM6UMXEFtixOi9bgdwW64PVCbNsF4bFSvD4afnCAZi6?key=uF1lrlKcv2DMk5fHIqZRlSjX)

So that means, Android can forward packets across the networks. Fine.

Can I ping 192.168.144.11 ? Nope, does not work.
My computer needs to know that 192.168.144.0/24 is behind 192.168.144.11. No worries, I am on a Linux computer.
`
sudo ip route add 192.168.144.0/24 via 192.168.4.21
`

And it is now pinging 192.168.144.11 ! Great. But not 192.168.144.25 … close enough.

But that isn’t a working solution

## New Solution needed ?

I can't dig deeper than this due to lack of knowledge on Android and permissions. But the issue is likely that we can reach the camera but it doesn’t know the ip routing to respond back …

On a Linux system, we would have done something like :
`
iptables -t nat -A POSTROUTING -o br-vxlan -j MASQUERADE
`

This makes traffic from 192.168.2.78 (my computer) appear as if it's coming from 192.168.144.11, ensuring replies are routed correctly.

No root, no route.

Any android expert to help the manufacturers to solve this on their default images for the greatest good ?

# Workaround

I don’t have a full solution but a workaround. As we know it is just a routing issue, we can just add a new proxy. Well that wasn’t simple … as we need a tcp proxy.

I used com.icecoldapps.proxyserver (not anymore on playstore, see alternative store https://apkpure.com/proxy-server/com.icecoldapps.proxyserver)

After install (see doc how to install apps). I did this:
![|602x802.9542277506982](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeyGj6bIvdsOuEY7FJQ1IdqlBbw9DMxHp8R2niEKy-2VAKiT47_r4p2HK-mQRpt9XpfuTC14FCH_nm6N_kzJlRuxMh3jpk8vFUGJE27REwr97hqzX-HoJFLWkNhWK-OhX4o8sa8Ow?key=uF1lrlKcv2DMk5fHIqZRlSjX)

![|602x802.9999999999999](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdfYo9oNDPDGbF98xi1wRZ-L1CWSdiNw1bHpUGYbr7UxlkyHRU9lMlQNghVlawI95ZkFrWcbb3pQhTYe7ReYWDg7iQd3y0XHDZVfUazvkjiLCBv7NQh0ihCOZ96QNz9ZV-f2BnXjw?key=uF1lrlKcv2DMk5fHIqZRlSjX)

That means that any tcp packets on the remote controller port 9090 will be routed to 192.168.144.25 on port 8554, and on the other way, the camera will send from port 8554 to the remote controller port 9090 that will forward to whatever connect vlc, any viewer, etc.

(you can choose whatever port you need, not necessary 9090)

Working picture :

![|602x451.00000000000006](https://lh7-rt.googleusercontent.com/docsz/AD_4nXezVMIVavw33vyAoq8dCVcIZKX0RTJReJKI0adkVCwg6u8YH8lJ4YvuTzhgvuKR_L7z4bdBPqqULdRl80Pjj8mNG5BaQ0Agj0JYbu5qN2LwmPYjoHki4J6uPV_lwyD5F5Rfw-MjnA?key=uF1lrlKcv2DMk5fHIqZRlSjX)

# Conclusion

You have now a solution to connect your Integrated Remote Controller to your own Wifi and share it video. This also means that you can connect multiple controllers on the same Wifi and have all their video accessible at same time !

If remote controllers manufacturers want to use this as a basis to fix the routing to allow full access to the IP devices connected on the Air Unit, that would be great. Hardware is great, just need some tweaks on software. Another solution would be to give some configuration on a proxy app like the one that is present on Herelink but works only with an HDMI camera.

My workaround is working and the app is doing a good job with a lot of great features, but doesn’t exist anymore on google store. If somebody more used to Android wants to propose a better version, develop a new proxy for the remote controller, feel free to share it !

