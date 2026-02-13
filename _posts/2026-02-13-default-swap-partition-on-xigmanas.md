---
layout: post
title: Default swap partition on XigmaNAS
date: 2026-02-13 17:24 -0500
reading_time: 3 minutes
tags:
  - BSD
  - FreeBSD
  - XigmaNAS
---

When installing [XigmaNAS](https://xigmanas.com/) (using the standard _embedded_ installation method), there are broadly two options available: installing **with** swap and data partitions, or installing **without** swap and data partitions.

When you choose to include those partitions, the OS, swap, and configuration files are each given their own partition on whichever drive you choose as your installation destination. While this is very simple and convenient, it works against the fact that XigmaNAS will load the entire OS into memory on boot, which allows for using relatively unreliable drives (like an SD card) to be used for the OS without much worry about longevity, since they are essentially read-only. If swap is also on that drive, it could see a lot of writes, reducing its lifespan.

The installation option without swap and data will install only the OS (and boot) partition to the selected drive. After the installation, you can choose partitions on other, more robust, drives to use for swap and data.

If you poke around an install that included swap+data, you won’t find any configuration referencing the swap partition, but XigmaNAS will still find and use it. It won’t be listed in the `fstab`, there’s no `rc` script that mounts it, etc. It just sort of shows up.

If you make your own swap partition, you may find that it doesn’t get used quite as magically.

It turns out that the [`rc` script](https://sourceforge.net/p/xigmanas/code/HEAD/tree/trunk/etc/rc) that handles a bunch of system initialization specifically looks for a partition called `/gptswap/` (`gptswap=$(/sbin/glabel status -s | /usr/bin/awk '/gptswap/{print $3;exit 0}')`).

So if you’re handling the swap partition yourself on a XigmaNAS setup, don’t get clever with the partition label. Stick to the same `gptswap` that the batteries-included installation uses to take advantage of the default swap detection that is built in.
