---
layout: post
title: "Network Shenanigans in Consumer Grade ISP Land"
tags: "Networking Silliness"
---

We've come a long way from the time I started poking around on computer networks outside a single system. POTS modems,
ISDN, T-carrier, Cable, DSL, and Fiber. My first connection to the outside world was on a white box IBM Compatible PC
circa 1991 on a USRobotics Sportster 9600. Yes, _back in the day, serial ports were fast_. As fast as your connection to
the outside world was...

Now we have GPON networks delivering up to symmetric gigabit speeds directly to your home. However, it is clear that the
equipment that we have in the networking space hasn't really caught up with the connection speeds. At my house, we've
been upgraded from the copper loop carrying VDSL to GPON fiber, along with coaxial cable from the cable company.

You might not even call it an upgrade, since the fiber had been installed when the neighborhood was first wired up,
thirty years ago.

But BellSouth gonna BellSouth.

For the DOCSIS crowd (and to a lesser extent, the DSL crowd), this doesn't really affect you as much, mainly because the
speeds aren't at the level that GPON is capable of.

### The Point at which the Issue Becomes A Problem

However, for the lucky few who are able to get an FTTH deployment, this is rapidly becoming an issue/problem for two
reasons:
1. Trust in the devices being provided (along with the services...I'm looking at you, AT&T DNS "Internet Preferences" tracking)
2. Security of the devices being provided
3. Performance of the devices being provided

Trusting the devices provided is really a question you have to answer for yourself. Unless you are a hardware engineer,
you are going to have some level of trust in *someone* who is providing the equipment.

If you want something to really keep you awake at night, BootHole as an (sort of, but not exactly) incarnation of **The
Ken Thompson Hack** is a good start. Reading _Reflections on Trusting Trust_ should scare you even more.

But even laying the trust and security issues aside for a while (Imagine having to write your own code in machine op codes or making
a rudimentary assembler on a MOS 6502), we can come to the practical matter at hand.

#### Performance is Actually Hard

The slowest computer I've used is an Apple ][e with a 1.023 MHz MOS 6502. I only say slow though in comparison to what
exists today. Steve Wozniak is an absolute genius for pulling off what he did with the Apple ][ (The same thing, of
course, could also be said of guys like Bil Herd and Dave Haynie at Commodore). Today, we're spoiled
with multicore and multithreaded processors that run circles around, well, everything made in the 1970s.

Granted this is fantastic performance way beyond what was thought possible in the 1970s, but while clock speeds went up
on chips, and we saw faster interconnect speeds for hardware, things not implemented in hardware came to rely more and
more on the processor to make up the difference in speed.

So in the Ethernet world, this means routing and packet switching. While you won't need the performance of a Juniper PTX
for your routing needs today (115.2Tbps forwarding capacity), you will want to make sure that your equipment isn't the
bottleneck on your network.

#### A Practical Application

In my case, I have fiber at my home, provisioned to symmetric gigabit speeds. The customer premises equipment
assigned to me is a Pace 5268AC. This is a combo VDSL2 and Ethernet router, which the provider helpfully calls a "residential
gateway". The box has a 4 port 1000BaseT Ethernet switch along with a separate 1000BaseT uplink (labelled ONT Broadband).
For VDSL2 customers, the box also has a VDSL2 broadband port. 802.11 radios are present in the 2.4GHz and 5GHz bands,
supporting up to 802.11ac. The main processor is a dual core MIPS chip from Broadcom running at 400MHz per core.
Official performance specifications in terms of switching throughput or routing throughput are not given.

So far, the 5268AC has worked pretty well. Not amazing, but decent. It passes the `Grandma Tilly` test, given the ISP's
predilection for having everything managed via their web portal, rather than on the device itself.

Routing performance is perfectly acceptable for what could be considered "typical home use" of email, web browsing, and
watching videos. Once you start to exceed the envelope, typically from the switch from wireless networking to wired
Ethernet, then it starts getting interesting. Line speeds on a typical 1Gbps GPON installation are about 940Mbps each
way, accounting for overhead. If you have one device using the connection, the router is capable of performing at that
speed.

That's not what happens though in a practical deployment. The ISP also offers a triple play over fiber, and offers the
not-well-advertised option of receiving TV over the fiber connection (as opposed to their preferred alternative). All
the TV streams will also take bandwidth to deliver, and the routing is a bit more complicated than a typical TCP connection
to an HTTP server.

This causes performance to choke when handling more complex scenarios. For us, we likely then would look to have a
cascaded router setup where we bypass the routing core, and move traffic directly to a device with a bridge. Unfortunately,
that's not possible either, due to the firmware installed on the router. Even DMZ Plus mode still routes traffic through
the routing table.

In addition, a previous firmware update in 2018 introduced a bug where bandwidth on DMZ Plus mode would be limited to a
fraction of what the provisioned line speed is. In my case, this meant I would achieve 150/150 on a 940/940 connection.
An 84% decrease so I could have the privilege of making sure my devices were secure.

We'll take a look at how I went about fixing this issue [in part 2]({% post_url 2020-08-18-network-shenanigans-part-2 %})