---
layout: post
title: "I've retired my Drobos"
tags: "NAS Storage Data"
---
At one point, I was a big proponent of Drobos. I actually own two, second generation units (back when FireWire 800 was a thing).
They've done well enough, I suppose, but having two of the boxes on my desk, while the rest of the stuff competed for space
on my desk was too much.

As a parenthetical aside, monitor arms are some of the best things I've ever done. If you have never experienced working
with monitors on VESA arms, I highly recommend giving that a try. Maybe I'll do a list of "Essential Equipment For A Software
Engineer" at some point in the future. We'll see.

So back to data storage. The Drobo units had a certain appeal to them, specifically around the Mac-like ease of use (Just
plug it in and insert bare disks and the system will configure itself for you to set up storage volumes) and the lack of
requirement of knowledge of how to configure disk arrays (needing the same size disks, arranging mirror and parity sets, blah blah blah).

But as is well documented elsewhere on the Internet, these units have certain drawbacks. One is the fact that in order to
see the log data from the units, you have to open a support case with Drobo support, as the logs are encrypted. This is
antithetical to decent practice for system administration. No one uses systems in Serious Businesses™ that operations staff
is not able to diagnose issues that arise on the front line. "But," I hear you saying, "I'm just a home user, so my use
case is not Serious Business™." This may be true, but it also means that you are still dependent upon an outside group,
and cannot really be self-sufficient.

All of your objections will go out the window the first time that you are not able to access your vacation photos from that
trip to Peru three years ago...you know, the one where Auntie Lois had the priceless expression on her face after accidentally
falling into the pool at the hotel after coming back from Machu Picchu.

You'll look at the Drobo Dashboard, and it will tell you that everything looks all dandy. Your computer may even show you
that the volume is mounted, but alas, you will not be able to open your Lightroom catalog or Apple Photo library. You'll
open a support case, and wonder if anything can be done.

But you, my friend, have an older Drobo. It's out of warranty, and now for the company, out of support. You can't even migrate
your disks to a newer Drobo. You'll be told by other tech folk to scurry eBay for the one model you can migrate to, and to 
get off a proprietary solution as quickly as possible.

I won't even touch on the fact that the model you have uses an interface that hasn't been included natively on Apple machines
since about 2012-2013 (and the fact that the Thunderbolt 2 to FireWire adapter is rubbish), and your only option on non-FireWire
machines is USB 2.0...

---

The reason for all the description above is because I've been rethinking my storage strategy and data backup strategy.
Time Machine is a wonderful piece of software that has lowered to cost to data versioning drastically and conditioned an
entire cohort of Mac users that "Backups are good, backups are important, backups are Easy To Do on a Mac." Time Machine
backups normally run on a direct-attached HFS+ volume (until Big Sur, when APFS is finally supported) and also on sparse
image bundles when backing up to a network location, like an Apple Time Capsule (NAS for the Mac masses!). But the Apple
AirPort line of products is no more.

But a NAS device was a good solution for my dilemma. And I'm also becoming a lazy sysadmin as I've aged, so while I could
have thrown together an UnRAID or FreeNAS ITX box, I was more partial to a Synology or QNAP box that had most of the main
features already buttoned down. I also wanted something that would have some expansion capability and be able to host a Plex
server.

Honestly the PCI Express slots on the QNAP were what sold me on the QNAP TVS-872N. Eight bays, a Core i3 15W processor with
Quick Sync Video (for transcoding), and if need be, a PCIe 3.0 x16 slot and an x8 slot. I bought four Seagate 4TB IronWolf
drives to put into a RAID 1+0, and started copying data off the Drobos.

The data copy took 6 days. No, that's not a joke, it really did take that long, across two devices, connected by FireWire 800.
Average transfer rate was about 10MiB/s. After the first enclosure finished, I repurposed the disks, and connected them up
to the QNAP to do some SMART tests on the existing drives.

Lo and behold, a 2TB Western Digital drive had failed SMART testing (almost immediately upon it being detected by the QNAP)
and a bad sector scan. This was a drive that the Drobo said was perfectly healthy, and had no issues with it. In fact, the
only indication that something was amiss was the fact that it went into "Data Protection Mode" three times during the copy
of data off the device. Besides that, there was ZERO indication that this drive was going bad, in any way.

That is reason enough to move my data off of the Drobo devices and onto something else.

---

If you cannot trust the devices that you have your data on, then what's the point of having the data? I'm going to be looking
at Backblaze B2 as my off-site solution for the first location, and then possibly an AWS or GCS archive for write only storage,
(AWS and GCS get you on data egress charges).

I'll write something about my thoughts on my QNAP box a little later on.