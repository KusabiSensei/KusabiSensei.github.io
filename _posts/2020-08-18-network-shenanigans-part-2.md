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
There is a great write-up [available here](https://github.com/MonkWho/pfatt).

The short version of this is that there are two pieces that have to be taken into consideration by the router. One is the
fact that authentication to the network is done via EAPoL (Otherwise called IEEE 802.1x), and the other is the fact that
traffic bound for the ONT requires an 802.1q VLAN ID to be set to 0 (Sometimes seen as Cisco priority frames).

Both of these can be handled by the use of the `netgraph` functions in the FreeBSD kernel. This allows for the creation
of a graph-based description of the virtual networking devices, and filtering and routing at the packet level.

In essence, we are actually creating a Man-In-The-Middle attack between the 5268AC and the ONT. This is because we require
the certificates that are present in the ROM of the 5268AC to be presented to the authenticator on the other end of the
optical network. Once the EAP Authenticator sets the port status to `Authorized`, normal traffic (including host
configuration) should be passed to FreeBSD by a virtual network interface. This can then be used by the routing
processes as the destination for outbound traffic from the LAN.

So the first order of business is to consider the cabling of the devices. This requires three physical interfaces,
although I used a fourth one for a separate LAN segment for multicast IPTV set top boxes. 

In this case, the ONT is connected via Ethernet to interface `em0` on the device, and the 5268AC's ONT Broadband port is connected to `em2`.
The primary LAN segment is connected to `em1`, and the IPTV set top boxes are connected by a switch connected to `em3`. Pardon the terrible ASCII diagram.

```
----------     ----------  ---------    --------   
| 5268AC |     |   ONT  |  |  LAN  |    | IPTV |
----------     ----------  ---------    --------
   | ONT            |      |             |
   |                |      |             |
   |---------[em2] [em0] [em1] [em3]------
```

The other piece of information we need to get started is the MAC address of the ONT interface on the 5268AC. Since that's
on the data plate of every single router, we have what we need to get started.

If you want to be really fancy, and you have the certificates from your gateway available, then you can do the 802.1x
authentication yourself, and you wouldn't need to have the gateway remain connected. I, however, am not that fancy.

### Packet Routing For Fun

So now we have to set up the netgraph in the FreeBSD kernel. Fortunately in the latest versions, netgraph is already
loaded in the kernel, so there's no additional kernel level work to do. In addition, thanks to the hard work of many
individuals already, there is a script that is able to do almost all the heavy lifting. 

In short, this script is configured to run at the start of boot, using an `<earlyshellcmd>` in the `config.xml` file.
This creates the netgraph, which bridges the correct interfaces together. In addition, it sets up the filters for EAP
traffic to route to the correct interfaces, along with setting the netgraph's virtual interface's MAC address to be the
gateway's ethernet address.

Once this happens, then we are able to take `ngeth0` and use that as the WAN interface of the router.

### IPv4 and IPv6

So once the EAPoL packets from the residential gateway are bridged over to the ONT, and the responses come back, then
standard traffic is able to pass. DHCPv4 is, of course, supported, and that's fairly straightforward. Simply map out
the RFC 1918 addresses for your local LAN, and off you roll.

IPv6 is a bit more of a challenge, and it will depend on your provider and their configuration. In my case, the provider
provides a site-local address via DHCPv6 and also will delegate a prefix for subnets to be assigned by the router. This
requires that the DHCP Unique Identifer be set appropriately. The provider works best with the DUID set to EN mode, the
IANA Enterprise Number set to `3561` and the identifier set to a value that can be calculated from the serial number of
the residential gateway [using this script](https://github.com/MonkWho/pfatt/blob/master/bin/gen-duid.sh).

The prefix delegation request needs to have the netmask of the requested prefix set to `/60`. This gets the assigned
prefixes, and allows you to then configure the router to assign `/64`s to different interfaces. Given that a standard
IPv6 subnet is a `/64` and contains 18,446,744,073,709,551,616 addresses, I don't see you running out any time soon.

### IPTV and Multicast

My provider, in addition to data service, also provides TV service over IPv4 Multicast. The method by which this happens
is a bit strange, but it is [well-documented here](https://github.com/MonkWho/pfatt/blob/master/U-VERSE_TV.md).

First, the box requests a unicast stream from the headend. While it starts playing the unicast stream, it makes an IGMPv2
membership request to the multicast version of the same stream. Then after about 10 seconds, it will seamlessly transition
from the unicast stream to the multicast stream, which saves on bandwidth, especially in the case where multiple set top
boxes are tuned to the same channel.

This requires that the firewall be set to allow traffic to the multicast networks between any interface. This is done
through the use of a floating firewall rule. In addition, it's necessary for your router to support proxying IGMP traffic,
so that subscription requests and such can be made of the headend to send multicast traffic to the box.

### Results

So far, the small box has done well for itself with no major issues cropping up. Granted, I haven't used IPSec or OpenVPN
with it yet, but we'll see how it does once we get there.

I accomplished the following major things with this small project:
* I increased packet routing performance
* I'm using equipment that I have root level access over
* I've closed down all ports from the WAN interface, including those that are open on the residential gateway by default

Next time, I'll get into how I've deployed DNS, since I have my own domain name (naturally...that's where you're reading this).
I'll also take a look at encrypting management traffic on various devices using certbot and Let's Encrypt.