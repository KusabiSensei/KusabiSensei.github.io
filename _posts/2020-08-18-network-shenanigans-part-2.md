---
layout: post
title: "Network Shenanigans Part 2 - Fixing It"
tags: "Networking Silliness"
---

[In part 1]({% post_url 2020-08-17-network-shenanigans %}) of this saga, we took a look at how ISPs haven't been able to
keep up with the technology and line speeds that they are providing. So for this part, we are actually going to fix this
problem, and also increase our network security.

### Finding a Router

I could have gone with a basic consumer router, or even a higher end consumer router, but I still would have concerns
about the source code. In general, something for which source code is available is a better choice (to me) than something
for which source code is not available.

So we'd be looking at something about in the league of a Asus RT-AC3200 and up. If you haven't done any network equipment pricing
recently, you'd also have a bit of sticker shock when you see that what is supposed to be a home router is actually US$250.
If you get into the 802.11ax equipment, you could be looking at up to US$450 for a router.

For kicks, I decided to see what kinds of single board computers were available. Turns out there are plenty of Bay Trail,
Braswell, and Kaby Lake SoCs made below 15W TDP, which means there's also a good chance that the cases could be fanless.

I picked up a machine from Protectli for about $400 which has a Kaby Lake Celeron 3865U clocked at 1.8GHz with two cores. There are
Braswell based systems that also are clocked similarly, but have four cores. They are also limited by the number of PCI
Express lanes available (4 PCIe 2.0 on Braswell, 10 PCIe 2.0 on Kaby Lake). Kaby Lake also support DDR4 RAM, so any expansion
should be easier than the DDR3L RAM used in Braswell and Bay Trail. 

More importantly for our use case, it has **six** Intel 82583V chips on the board. That's pretty good.

Protectli ships their systems without an operating system, although their website mentions that they can install an OS
upon customer request. Heck, I would have been fine with the barebones configuration, and sourcing my own RAM and mSATA
drive.

I decided to give pfSense® CE a try to see if it would be able to do the job. The installation went smoothly on the device,
using the included serial console cable (and an old Gateway Profile 6, which is my only computer with a serial port built in).

### Understanding the ISP's Networking

So before configuring the router, we first need to understand a little bit about how things are set up for the ISP.
There is a great write-up [available here](https://github.com/MonkWho/pfatt)