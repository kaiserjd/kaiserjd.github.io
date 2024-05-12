---
title: Defeating CGNAT With Wireguard
date: 2024-05-11 16:00:00 -0500
categories: [Networking]
tags: [it, networking, wireguard]
img_path: /assets/img/
---

# Intro

I've been having a problem with my self-hosted servers for the past year or so. I tend to host game servers for myself and a few friends, and it they worked flawlessly up until about a year ago. It wasn't until I cobbled together my own [OPNsense router](https://kaiserjd.github.io/posts/OPTIPLEX-AS-ROUTER/) that I figured out the reason: my ISP had put me behind carrier-grade NAT (and hadn't even bothered to give me IPv6!).

To be honest, I probably could have just asked to be given a dedicated IPv4 or v6 address, but where's the fun in that?

# An Overview of My Setup

This whole ordeal was certainly 'fun'. I may come off as a little sour, and while I did have a lot of headaches with this setup, I learned a lot and most of my issues were self-inflicted anyways. The payoff was definitely worth it, though.

My setup is mostly based upon mochman's popular method of [bypassing CGNAT](https://github.com/mochman/Bypass_CGNAT) with Wireguard. However, the fact that the automated installer didn't help me much and due to my choice of VPS (Oracle cloud), I was forced to do everything manually.

My setup goes something like this:
![Setup](CGNAT-bypass.drawio.png)

Nothing too crazy or different here. Where the real pain points were was in the IPTables and Wireguard configurations.

# Home Network Side

Setting up my home network was by far the easiest part of this ordeal, and probably took only about a day or two. A large reason for making my own router was for access to VLANs, and as you can see here, I created a DMZ for anything that is public-facing. There's plenty of information around on setting these up, so I won't describe that process too much, just keep in mind that it is a factor in my setup.

The NGINX configuration was also incredibly straightforward, and worked on the first try. Currently I am using my setup to host game servers, but NGINX is more than capable of keeping up with this kind of traffic. I also set up a UDP voice server after my main setup was complete, which took a few days of troubleshooting, but that's another story.

# IPTables and Wireguard

This is the actually interesting part of my setup that took the longest to get working correctly. As was shown in the diagram, any user that wants to connect to a game server (in this case) would use my VPS' public IP - or in my case - a SRV & A record pointing to that IP. I do all of my firewalling on this machine, utilizing both Oracle cloud's firewall as well as Ubuntu Server's built-in solutions (Iptables, which I'll get to in a moment).

On both this machine and my NGINX machine are installations of Wireguard, with my VPS being the server and my NGINX machine being the client. After ensuring NGINX worked correctly, Wireguard was my second focus, and oh boy did it take quite a bit of focus.

Apparently, Oracle has some very specially configured Ubuntu Server VMs which require some special workarounds. This is also one of the reasons Iptables is required for this setup: special NAT and firewall rules are required to forward traffic over a Wireguard tunnel.
My rules, which were simplified from [this](https://www.reddit.com/r/WireGuard/comments/oxmcvx/comment/h7nl24o/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) glorious Reddit comment:

```bash
# Workaround for Oracle Cloud shenanigans
PostUp = iptables -t nat -I POSTROUTING 1 -s 10.0.0.0/24 -o ens3 -j MASQUERADE
PostUp = iptables -I INPUT 1 -i wg0 -j ACCEPT
PostUp = iptables -I FORWARD 1 -i ens3 -o wg0 -j ACCEPT
PostUp = iptables -I FORWARD 1 -i wg0 -o ens3 -j ACCEPT
PostUp = iptables -I INPUT 1 -i ens3 -p udp --dport <WG-port> -j ACCEPT
```

If you also wish to use these rules, keep in mind that these are a part of my Wireguard .conf file. Also make sure to include the relevant PostDown rules so you don't duplicate any each time your Wireguard interface is brought up.

After the tunnel was connected and working, it was time to direct any incoming traffic into that tunnel. Fine-tuning these rules can be quite tedious, but after learning how Iptables works it is not nearly as daunting.

First are the rules to actually allow traffic on specific ports into the machine, duplicated for both TCP and UDP.

```bash
# Allow traffic on specified ports
PostUp = iptables -A FORWARD -i ens3 -o wg0 -p tcp --match multiport --dports <ports> -m conntrack --ctstate NEW -j ACCEPT
PostUp = iptables -A FORWARD -i ens3 -o wg0 -p udp --match multiport --dports <ports> -m conntrack --ctstate NEW -j ACCEPT
```

Yet another excellent [post](https://wickedyoda.com/archives/956) I found gives a very good explanation for how to forward traffic between the Wireguard and ethernet interfaces.
Some of the rules I have look something like:

```bash
# Allow traffic between wg0 and ens3
PostUp = iptables -A FORWARD -i wg0 -o ens3 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
PostUp = iptables -A FORWARD -i ens3 -o wg0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward traffic from ens3 to wg0
PostUp = iptables -t nat -A PREROUTING -p tcp -i ens3 --match multiport --dports <ports> -j DNAT --to-destination 10.0.0.2
PostUp = iptables -t nat -A PREROUTING -p udp -i ens3 --match multiport --dports <ports> -j DNAT --to-destination 10.0.0.2

# Forward traffic back to ens3 from wg0
PostUp = iptables -t nat -A POSTROUTING -o wg0 -p tcp --match multiport --dports <ports> -d 10.0.0.2 -j SNAT --to-source 10.0.0.1
PostUp = iptables -t nat -A POSTROUTING -o wg0 -p udp --match multiport --dports <ports> -d 10.0.0.2 -j SNAT --to-source 10.0.0.1
```

There are almost certainly better ways to do this, such as including variables for things such as TCP/UDP ports, but this works and it isn't too annoying to edit if necessary.

Again, if you'd like to copy these, make sure you include the relevant PostDown rules in your Wireguard configuration file.

# Conclusion

This whole process probably took me about a month of on and off work. I left quite a bit out and would highly recommend again to anyone who has this need to try [mochman](https://github.com/mochman/Bypass_CGNAT)'s automated solution before going this route.

While I may not have had a lot of time to dedicate to this project over this past semester, and while I certainly made plenty of mistakes along the way, I learned an absolute ton from doing this. I'm almost a little bit thankful to my ISP for not making it easy on me, as I wouldn't have had a reason to learn this otherwise.

Just kidding. I honestly can't believe Armstrong is still in business. CGNAT and no IPv6 in today's world is absolutely ridiculous.
