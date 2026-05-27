+++ 
draft = true
date = 2026-05-15T01:36:24+10:00
title = "The $30 Arm Proxmox Backup Server"
description = "Taking too many chances at once."
authors = ['Timothy']
tags = ['article','linux']
+++

*Taking too many chances at once.*

I could not resist allure of creating a cheap, low wattage Proxmox Backup Server. I detail my thoughts and provide a short guide to follow along.

### Why would I want a Backup Server?

I've never been good at keeping backups of my homeserver, and I've always taken issues with my deployments as a chance to rebuild and make them better. There are times however when I just want to keep a service deployed, particularly one which has several custom, hard to automate tweaks that keep service(s) running.

A backup server gives me some peace of mind knowing that I have a copy somewhere that is relatively up to date and easily deployable back to the hypervisor should I require. I don't need to faff about with cold backups, and I'm dealing with a host that is physically sperate from my hypervisor. In the extremely rare event that I'll need to start from scratch that may also come in handy.

As with most things that could affect you down the line, but doesn't at present, the easy option is to bury your head in the sand and wait for disaster. I'll try to be proactive instead.

### Re-using Dormant Devices

I didn't originally plan to set up a [Proxmox Backup Server](https://www.proxmox.com/en/products/proxmox-backup-server/overview) (PBS) using the hardware I did. However, my perusal of the [r/homelab](https://www.reddit.com/r/homelab/) Reddit page inspired me to utilise several components I had laying about and combine them. My newest technological concoction contained:

+ A spare 1 terabyte 2.5" Hard Disk Drive (HDD) from an Xbox One X.
+ An HDD enclosure previously used to back up old (now irrelevant) files.
+ An [Orange Pi Zero 3](https://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-3.html) with 2 gigabytes of memory from an early attempt at a [k3s](https://k3s.io/) cluster.

The Orange Pi was the obvious choice for such a server, given that it's low power and about the only thing I've got to tackle such a task. I had originally bought it to learn k3s on hardware and although the experiment worked well enough the relative lack of horsepower meant I never really could deploy services onto it. Looking back, the $30 I paid for it was a bargain, something perhaps never to be matched again with the direction hardware prices are heading.

Although fairly unassuming, it's more than reasonable to run a backup service as it's load is periodic in nature. More importantly for my use case the second and subsequent backups of a service is always deduplicated, one of among many of [PBS' useful features](https://www.proxmox.com/en/products/proxmox-backup-server/features). This can save utilising unessecary network and storage capacity. The Gigabit Ethernet port of the Pi would be adequate for my needs.

The secret ingredient would be the requisitioned HDD attached via USB to the board. A terabyte's worth of storage is everything I'd need or more than for the services I intend to backup. I also designed some rails for the Pi to slide onto the enclosure using the same design as from my Pi cluster [dicussed in brief here](posts/learning-freecad-engineering-principles/#starting-with-cad-and-the-first-hurdle). Some printing and assembly later, and it was ready to go.


{{< inset-img src="images/opi-chassis.jpg" alt="Photo of an Orange Pi with heatsink attached to an external 2.5 inch HDD chassis with cables connected. " caption="*Orange Pi complete with HDD and mounting brackets.*">}}


### But can it Run on ARM?

One of the biggest hurdles is the lack of official support for PBS on arm. This, alongside the countless iterations and configurations of kernels available for devices using the arm instruction set mean that compatability in general is touch and go, especially if you're extending or using features close to the kernel.

Arm however provides us a very significant advantage over the more traditional x86 instruction set, power usage. To use arm for a small backup server operating infrequently and with modest hardware requirements is almost a given. There aren't many boards that

To actually deploy PBS, we have to solve two major hurdles that exist as a result of this instruction set choice:

+ We must use an up-to-date OS compatible with PBS.
+ We need to find or build PBS packages compatible with arm.

On top of that, we will probably need to resolve minor issues that show up such as missing packages or interesting defaults. All in good time.

As PBS is built ontop of Debian, derivative distro [Armbian](https://armbian.com/) is a natural choice. It provides several builds, including several desktop environments

Armbian

Proxmox on Arm
https://github.com/wofferl/proxmox-backup-arm64


{{< inset-img src="images/opi-wattage.jpg" alt="A view of a power meter displaying 4.6 watts of power usage of the Orange Pi." caption="*Miniscule power usage on the Pi.*">}}

### Installation Guide

Raspbian - RPi Page Size Limit

prereqs:

To get the full experience of Proxmox Backup Server, we'll need to build it ourselves so that we can use ZFS. If you're not interested in using ZFS, you can use a precompiled armbian build for your board.


./compile.sh BOARD=orangepizero3 RELEASE=trixie BUILD_DESKTOP=no BUILD_MINIMAL=yes KERNEL_CONFIGURE=no INSTALL_HEADERS="yes" ENABLE_EXTENSIONS="zfs"


ifupdown2 package






### What could Possibly go Wrong?

In the final analysis, a backup server must be reliable. I have not done that and I am nowhere near close to matching the several nines provided by cloud providers. In fact, I have probably ran from that principle as quickly as possible. Here are some of the reasons as to why:

+ The boot drive of the Orange Pi is an SD card, and is bound to fail sooner than later with enough writes.
+ I have a single disk with no parity for the backups I create. This may (or given enough time will) prove to be an issue.
+ I still don't have an off-site copy of my data, and it's not on a distinct medium. I fail the 3-2-1 backup rule spectacularly.
+ I am running unsupported software on an unsupported Operating System (OS).

As you can see, this is a recipie for eventual disaster. I am keen however to see how long this service remains operational, and am invested in keeping it going in the event I may need it's contents.
