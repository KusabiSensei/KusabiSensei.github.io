---
layout: post
title: "Networking and Thoughts on Competence"
tags: "Networking Silliness Competence"
published: false
---

I don't seem to be able to go that far along without running into some issue that just completely ruins the flow of work
or ideas. Then again, if you have read Jeff Atwood [on why can't programmers program](https://blog.codinghorror.com/why-cant-programmers-program/)
, this should not be surprising in the least bit.

In the case of American ISPs, this seems to happen on a basis that should really be embarrassing to the ISPs. But it's not,
mainly due to the fact that Aunt Tillie and Grandma Mary just want to be able to get some email and see pictures of their
family.

Or so that's what the customer profile of the 1990s told us was the case.

It's amazing how the rest of the world has found new uses for moving bits of data to and from residences, and leveraging that
into the amorphous "business value." Well, not really. There are always going to be new things that will have to be done
over a data connection, and consumers will want them.

Now we have 4K video streaming from providers, we have video conferencing going on all the time, we have VPN connections
flying bits of data back and forth to companies. This was obvious to those of us who were working remotely full time prior
to March of 2020, but since then, it's become The Standard Way To Work™. Most of our shared residential bandwidth is allocated
to the downstream direction. That's to be expected, but now we are sending data out to the network as well.

Makes you wonder why Comcast doesn't actually advertise their upload speeds. "We have Gig speed!" the marketing department
tells prospective customers, but won't tell them that because they are a DOCSIS network, the upload speed is actually 35Mbps.

Yes, the fastest upload speed sold over a coaxial ISP link is 35 megabits per second. Yes, Comcast is saying they could do
symmetric provisioning, and trialled it at one home in Jacksonville, FL. No, it still doesn't hold a candle to fiber to the home.

That doesn't mean that FTTH is amazing in the US. We still have US ISPs to deal with. And for FTTH in my neck of the woods,
that means AT&T. I'm not lucky enough to have an EMC providing service, nor Google Fiber.

#### AT&T and Networking Shenanigans

[As we've discussed before]({% post_url 2020-08-18-network-shenanigans-part-2 %}), AT&T has certain requirements for accessing their fiber network. The most annoying one is the
requirement that you use the AT&T provided router to connect to the optical network terminal. For most customers, this is fine.
However, with the rise of more common work from home, we need to be able to support more connections and states on the firewall,
given that this is generally limited on most CPE for whatever the reason.

This works well enough, right up to the point where you get an edge case. Like the CA certificate for signing CPE and headend
certificates expiring (as any updates to that certificate are delivered by firmware update).

So when the CA certificate expires, this means that the 802.1x EAPoL handshake will fail, which means that you can't get on
the network. It also has the side effect that the gateway can't actually have an update pushed out to it by the headend.

Oops. The best thing that came out of this was the fact that I was able to get through the AT&T technical support tree in
under a minute, and got it to schedule a next day appointment.

A full reset of the 5268AC to factory ROM version (hold the reset button on the back for 45 seconds) allowed it to pull the
latest update, which restored service about 30 minutes before the tech arrived. But the tech also had some BGW210s on the truck,
so he swapped the 5268AC out for a BGW210. Score.

#### Reinstalling the Router Software

No good deed goes unpunished however, and my router box (The Kaby Lake Celeron U box) decided that it was going to throw
some read errors on the mSATA drive. While refreshing the Let's Encrypt certificate for the web interface, the FreeBSD kernel
crashed with a page fault, and caused corruption on the disk, leading to inability to boot.

One afternoon of system recovery later (thank God I was able to grab the configuration file), it's back up and running.
It did give me a question that I will try to answer.

#### Why Do We Have a Lack of Competence?

To be fair, this is the wrong question to ask. For most people, who aren't technically involved, having someone be able to
walk them through a power cycle or basic troubleshooting is actual competence. But for highly technical users, this is an
exercise in frustration. The worst thing you can do for technical users, is run them through the ringer of front line support
("Have you tried turning it off and back on again?")

In my experience, this seems to be something where I get an actual issue, that I know requires a response from the company,
call in, and have to go through the level 1 support script. (Yes, I've turned it off and back on. Yes, I tried another computer.
Yes, I've done a soft reset. No, I don't have the same issue if I use a different connection.) Unfortunately, there's no
easy way to skip past this and engage level 2 or 3 support. About the only way to do this is turn off the ONT and let it
decide that you need an actual technician onsite to look at the issue.

However, turning a router off and on doesn't solve the fact that there is no documentation that while your router is delegated
a `/60` IPv6 prefix, you will only get a `/64` from the router. Or how to deal with more complex routing and firewall tuning
scenarios, that normally would require an edge router.

But maybe this is because for those of us who have competence, having to go through things like this actually make us doubt
our competence at times.

I'll explore this, along with imposter syndrome, elsewhere.  