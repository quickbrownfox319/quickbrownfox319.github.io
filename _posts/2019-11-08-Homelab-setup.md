# Homelab

I've recently been getting more and more interested in the idea of working on my own internet infrastructure and having finely tuned control over how my network operates. There are a few tiers of how most people typically approach their internet:

#### 1. ISP provided equipment
This is by far the most common one that people do as it's the least amount of hassle. Typically, the ISP such as Comcast provides a basic plan to the consumer, usually with the option of renting the router/modem for a monthly fee. This makes the most sense if you only want to do basic web surfing and video streaming in a relatively small apartment with one or two devices. If anything goes wrong, usually a not-so-quick call to the ISP can eventually resolve the issue without additional fees.

#### 2. Buy your own equipment
For the brave, there's the option of buying your own equipment, as those monthly rental fees tend to add up. Most recent estimate for renting a router from an ISP was something along the line of $10/month for a modem and router combo, which adds up to $120/year. This is for usually pretty garbage units, causing people to (rightfully) complain about their terrible download/upload speeds that they're overpaying for. In comparison, you can buy your own modem and router for a little more than that, and own it forever. The hardware you pay for tends to be of better performance if you choose well, making the economics of buying your own equipment that much more attractive. You can choose to separate your modem and router into their own respective units, where management is usually set (and forgotten) through a clunky web interface. The web portal will allow you some basic setup features such setting the SSID, port forwarding, client management, and maybe even some advanced features like adding network storage or a VPN server.

#### 3. Build your own infrastructure
Homelabbing is basically taking point #2 above and beyond for micromanaging your network. It can be as simple as using a Raspberry Pi for a DNS server to a rack full of blade servers and managed switches. Just go look at [r/homelab](https://old.reddit.com/r/homelab/) for an idea of what some people go all out on. A lot of users tend to be network and infrastructure engineers who enjoy trying new things at home before deploying it work. Personally, I enjoy playing with virtualization and networking technology and wanted to make my network more managed and robust. For example, you can use a Raspberry Pi as a [Pi-hole](https://pi-hole.net/) for network-wide ad blocking. That's cool, but you need a fallback if it goes down. The solution I'm using is to host it on a VM on a server, which can quickly restore backups of a working configuration in case something goes wrong.

## My Lab
My current homelab is still a work in progress, but you can separate it into two groups: the virtualized and software side, and the hardware side.

### Hardware
Current hardware is a combination of salvaged and newly purchased. If you look online, you'd be surprised how cheap enterprise hardware can be. For now, my setup is home/small business level, but it'd be nice to one day have a rack server with a Dell R740 running in it.

  * Unifi Edgerouter X - Router and firewall.
  * Unifi UAP Pro - WiFi AP for wireless clients.
  * Netgear GS108 - Unmanaged switch that I salvaged from the trash.
  * Synology Diskstation, 2x3TB HDDs - Holds backups and ISO images for creating VMs.
  * Whitebox server i7-6700k, 1TB SSD, 32GB RAM - Hypervisor host for my VMs.

### Software/Virtualization
I was originally going to use XenServer as my hypervisor, but wanted to try an opensource alternative and settled on [XCP-ng](https://xcp-ng.org/), the opensource fork of Citrix's XenServer. Virtual machines I have so far:

  * [Xen Orchestra](https://xen-orchestra.com/#!/xo-home) - management VM to handle overview, creation, backups, etc.
  * [Pi-hole](https://pi-hole.net/) - DNS server for blocking ads. This is probably my favorite service I am running.
  * Unifi Controller - For managing the UAP Pro, allowing me to manage wifi clients, and finely tune the radios. In the future, additional UAPs would also be handled here.

The nice thing about virtualizing is that if things go wrong, I can restore the backups of my VMs to a point that worked before. By having my system perform monthly backups, I (hopefully) won't have to go too far back to restore my network to a working state. Additionally, everything is handled on a single machine. This is a two-edged sword, as on one hand I can easily move to a different location and get things going again pretty quickly. On the other hand, a physically damaged machine without offsite backups is a very bad situation to be in. The largest advantage to virtualization for me is the ability to self-host my own services and start moving away from relying on companies that control my data. Future services and features I'd like to implement in no particular order:

  * VPN - something I can remote back into my network securely, as well as safely surf the web from.
  * VLANs - currently working on this one, but it's good to segregate the network for both security and efficiency.
  * Plex - stream my own media.
  * Gitlab - host my own repositories and eventually move off of Github.
  * Nextcloud - selfhost my own cloud file storage.
  * [Malware aquarium](https://xkcd.com/350/) - besides being awesome to look at, honeypots to study malware in the wild would be pretty cool.

## Final Thoughts
There is still a lot of work needed, and I'm trying to implement best practices for security and management along the way. However, for the time I am glad I have the barebones of a homelab working to the point I can do basic tasks and access the internet.