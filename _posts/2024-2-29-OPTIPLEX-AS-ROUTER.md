---
title: Dell Optiplex as an OPNSense Router
date: 2024-02-29 23:00:00 -0500
categories: [Cybersecurity]
tags: [cybersecurity, it, networking]
img_path: /assets/img/
---

# Intro

As I was searching for a new hardware solution to install OPNSense to, I came across
a lot of discussion on using old business PCs for this purpose.

I couldn't find many detailed explanations of people's experiences using these computers as a router, so I figured I'd include my experiences and learning from doing this on my own network.

# The Build

Among the literature I did find discussing this prospect, I found a lot of questions on what specific devices to use. By far the most recommended were old Dell Optiplexes, specifically the Small Form Factor models for their full-sized PCIe slots.

I originally went the way of an Optiplex 3050 with an i3-7100, and an Intel I340-T2 as a NIC. I actually recieved the same model Optiplex with an i5-7500, as well as an Intel I340-T4 - both technically upgrades, but worrying with regard to power draw. However, with some preliminary testing, this device wasn't pulling more than 18 watts from the wall under average load (_with some power saving settings enabled in BIOS and PowerD on Hiadaptive_).

I'd like to do more testing in the future on the power efficiency of these computers, but for now it doesn't seem like much of an issue.

> A side note to anyone who wishes to prepare a similar setup: the I340s come stock with a full-sized mounting bracket, but the Optiplex SFFs have low profile PCIe slots. As such, it is important to purchase a low profile bracket _before_ your other parts arrive, lest you wish to be disappointed (I am _definitely_ not speaking from experience here).

![The Optiplex](optiplex-redacted.jpg)
_It's almost like it's meant to be..._

_Also yes, the NIC is a little.. off. We love him all the same._

I don't have much to say about the actual installation of either the parts or OPNSense itself. Both were easy to accomplish and went off without a hitch.

# The Unbound Debacle

While the _initial_ setup wasn't difficult at all, getting Unbound to work as a recursive DNS resolver was certainly not; at no fault to the OPNSense developers, it should be noted.

No, my mistake was a simple one. After first enabling Unbound and configuring a few settings, only the router itself would utilize Unbound as it's DNS resolver. Every time I'd check for updates, a gigantic spike of requests would appear in the graphic report, but nothing more. Every other device wasn't being assigned Unbound as their resolver. I'd be curious to know if anyone who reads this can figure out my mistake just from that description alone.

As I was setting up Unbound as a recursive resolver, I had set my upstream DNS servers in System > Settings > General. However, what I had neglected to realize was that, in my initial setup of the firewall, I had _also_ set DNS servers in the DHCPv4 service - completely overriding the system's default DNS servers.

Coming to the realization that my woes were caused by such a simple mistake was painful at first, but as soon as the DHCP leases began expiring and the sweet, sweet data started pouring in, it didn't hurt as much.

![Unbound Working](unbound-working.png)
_The moment I figured it out_

# Long Overdue

This project was definitely long overdue for me. There really is nothing like setting up a piece of software like OPNSense and tinkering around with it, and I can't wait to see what other shenanigans I can get up to with this thing.

If you have any questions or comments, as always please feel free to reach out.
