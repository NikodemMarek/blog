---
layout: post
title: "I need storage"
date: 2026-08-21
categories: self-hosting hardware software thoughts project
---
I've been moving to self-hosting most of the stuff I use daily, over to my own infrastructure. My current storage solution had several issues. So I decided that I need (want) a new bulk data storage. I've tried to document my whole thought and execution process here.

## Issues with the current setup

My current setup was meant to be more of a POC than a real solution. It is my old ASUS laptop, with an external USB SSD, with a capacity of 1TB. While the USB connection is not ideal, it was working fine. The real issue was network connectivity.

First, I had it connected over Wi-Fi, due to cable constraints, and the home router being mounted on the ceiling. There must be a problem with the driver or some other slow corruption going on, because after a few days of working flawlessly, it dropped the connectivity and reported a crazy log, which I did not bother to understand, as ideally I do not want my NAS to work over Wi-Fi.

So the next solution was to connect the laptop to another home router, via an ethernet cable. This was also very finicky, as the ethernet port is broken, and does not hold the cable in place very well. Anyway, after fiddling with the cable for a bit, the interface changed its state to UP, and I decided to give it a try for a bit, due to lack of better options. Also, Wi-Fi was still connected for redundancy.

And this was working fine for a bit, but then also died unexpectedly. And this is when I realized that the old laptop won't cut it if I want to access my data reliably.

## Possible solutions

1.  Rent cloud storage.

Cost:

With one of the cheapest storage providers, MEGA (10TB):

~90 PLN/month * 12 months * ~5 years = ~5400 PLN

Pros:

-   Always available.
-   (Theoretically) No size constraints, infinitely expandable.
-   I don't have to manage the hardware.
-   I don't have to handle the backups.

Cons:

-   Subscription (evil).
-   No guarantee that the prices won't go up.
-   I don't own the data.
-   Network latency.
-   Since I don't trust the providers, I'd have to manage encryption myself, which means a custom service that proxies the encryption, which means a maintenance burden.
-   "You'll own nothing and be happy".

Decision: I don't like subscriptions, the cost is high, considering it is just storage, without any other benefits, and on top of it, I have to provide some more infrastructure to silence my security concerns.

2.  Buy a pre-made NAS.

Cost:

Case: Terramaster is pretty much the only sensible option, as it is budget-friendly enough, and it is possible to install another OS on it easily (according to The Internet). Costs ~2600 PLN. It also comes with some extra compute, so I can expand my Kubernetes cluster onto it, and benefit from the closeness of the data.

HDD drives: A new 10TB Seagate IronWolf costs ~1800 PLN; however, recertified can be bought for ~1300 PLN, and I'm fine with taking the small risk. For redundancy, I need at least 2x, so around ~2600 PLN. While searching for the drives, I found 18TB Seagate IronWolf Pro drives for ~2200 PLN each, and decided to go with them for a bit of future-proofing. So the final cost was ~4400 PLN.

SSD drives: I need at least one SSD to hold the OS, 512GB would probably be enough, but since I'm buying it either way, some extra fast storage may come in handy. Samsung NVMe M.2 990 PRO 1TB is the chosen model, because I found it on a discount, for ~750 PLN.

Electricity: This may vary, but 1 kWh costs ~1 PLN in Poland. The power consumption of a NAS is roughly estimated at ~250 kWh/year. So the cost would be ~250 PLN/year * ~5 years = ~1250 PLN.

Total: ~2600 PLN + ~4400 PLN + ~750 PLN + ~1250 PLN = ~9000 PLN.

Pros:

-   I own the data.
-   I handle encryption.
-   One-time purchase (not counting electricity).
-   Somewhat expandable (2 bays for HDDs, and 2 slots for M.2 SSDs).
-   Low network latency on local network.
-   Extra compute power.

Cons:

-   Electricity may get more expensive.
-   I have to manage the hardware.
-   I have to manage the backups.
-   I have to deal with the downtimes.
-   I own another physical asset.

Decision: I chose this. It is not horribly expensive, has a prospect of working for many years, includes extra compute power which is already a bit lacking on triss, and most importantly, I own the data. If something forces me to, and it won't make any more sense financially, I can always sell it.

3.  Build a NAS.

Cost:

Similar to the pre-made one, I could probably get some components cheaper, get rid of some compute power (I'll mostly be using it as an NFS server), and pick ARM hardware for lower energy consumption.

Pros:

-   Same as pre-made option.
-   Small energy footprint.
-   More expandable.

Cons:

-   Same as pre-made option.
-   Power adapters, isolation, fire safety, etc. are difficult to get right.
-   Too many options (decision paralysis).

Decision: I don't have nearly enough experience with hardware to make the right
decisions, and it is not worth my time to pursue this knowledge.

4.  Don't hoard the data.

Pros:

-   Cheap.
-   Flexible.
-   Minimalistic.

Cons:

-   No future-proofing.
-   Limiting.
-   "You'll own nothing and be happy".

Decision: This was a good option; I could just do with some cheap cloud storage (<1TB), but this comes with all the disadvantages of cloud storage, and I'm already near 1TB of data, with no idea what to get rid of, and I've just started my data hoarding spree. While this makes sense financially, it is very limiting, and not future-proof.

5.  Fix the old setup.

Decision: C'mon, we ain't doing that.

6.  Store the data offline.

Pros:

-   Cheap.
-   Very expandable.
-   Almost no setup.
-   Low maintenance.

Cons:

-   No access flexibility.
-   Time-consuming.
-   Does not work well with my self-hosted services.
-   Just me, hard to share.

Decision: While this is viable, and I could get an external HDD, or an HDD with an enclosure, and transfer the data only to and from my own devices, it is not a very elegant solution. It does not really work well with my current infrastructure, which relies on constant data access, and introduces friction when accessing the data.

## The hardware

I have chosen Terramaster F4-425 Pro N305 with 8GB RAM; it satisfied my needs of being compatible with software other than the stock version, and it has 4 bays for HDDs, 2 of which I will fill immediately. It also has 3 bays for M.2 SSDs. Since I have no need for extremely fast storage (I'm looking at you, AI datacenters), I'll just fill one bay. The 16GB RAM, N350 version was tempting, however, RAM is pricey, and I can always upgrade later. Besides satisfying the requirements, it was on a 20% discount.

For the HDDs, I decided to go with Seagate IronWolf Pro series, as it is recommended for home NAS solutions, with a capacity of 18TB per drive. I was also considering Seagate Exos, as the price per terabyte is very cheap, especially for recertified drives, but honestly, I won't need more than 20TB for a while, and in the meantime, I could find some good discount for more NAS-friendly drives.

Additionally, I needed one M.2 SSD, and the decision was based mostly on the TBW rating (Terabytes Written), since the speed of the M.2 itself won't be a bottleneck for me. The two options were Samsung NVMe M.2 990 PRO and WD NVMe M.2 Red SN700, both of 1TB capacity. I ended up buying the former, just for the price.

## The software

As an immutable software enthusiast, I had to go with NixOS for my NAS, especially since I had a pretty nice and proven config for my previous setup (roach). The only change I've made was replacing the Btrfs filesystem with ZFS, which proved to be a bit of a challenge, but not in the scope of this writeup. I've created two pools: one for the whole SSD and one for both HDDs. The whole filesystem is encrypted with aes-256-gcm.

The first one is for the system itself, and is partitioned into `/boot` for the bootloader, `/nix` for the nix store, `/persist` for the files that won't get wiped on reboot (impermanence setup), and of course `/`. The decision not to have a swap partition or file was based on common problems other ZFS users faced with it. If I ever find that it is lacking, I can add it on ZFS and deal with the consequences, or by resizing the ZFS area and creating another ext4 partition just for the swap.

The second one is purely for data. The 18TB drives are mirrored for redundancy via ZFS vdevs. On top of it, I have created a few datasets, like I had on roach previously, but this is subject to change. Sadly, disko natively does not support updating the setup, so whenever I add a dataset or change something, I have to do it both in config and from the shell.

The main functionality of the NAS for me is to serve the data over NFS, so I have it set up exactly like I had before, exposing the data mounts to my tailnet. ZFS offers NFS functionality natively, but just for the sake of decoupling, I decided not to use it, and have an explicit, declarative definition using the Nix module.

Just like before, I also have a k3s server with Longhorn enabled on the NAS, connecting as a worker node to triss, over the tailnet, no nuances there.

One more problem to solve with this setup was the ethernet drivers, since the F4-425 has two Realtek 5GbE connectors, requiring `r8126` drivers, which are not yet supported natively by the Linux kernel. Fortunately, there exists a wonderful [realtek-r8126-dkms](https://github.com/awesometic/realtek-r8126-dkms) repository, and Gemini nailed the Nix package derivation on the second try, so it was just a matter of disabling the `r8169` driver and enabling the packaged one.

## Retrospective after time

TBA: The setup has been running for just a few days, so it is hard to provide any insights about it right now. I'll try to come back and update this section in a few weeks.

## Final thoughts

While a storage system does not seem that complicated at a glance, there is a lot of consideration to put into it. While it is definitely viable to reuse old, random hardware, it is definitely easier to use dedicated, purpose-built hardware. It is probably overkill and more of a hassle than a benefit for most people to have their own NAS; I was really considering going with cloud storage, and I would describe myself as tech-savvy and very experienced with dealing with various problems from this domain. So there is not really any reason for me to recommend anyone to go this route, besides data privacy and ownership, which barely anyone cares about nowadays anyway.