---
layout: post
title: Basic Network UPS Tools (NUT) and Home Assistant
date: 2024-02-19 14:38 -0500
tags:
  - Home Assistant
  - Smart Home
  - Network UPS Tools
  - NUT
---

I recently setup [Home Assistant](https://www.home-assistant.io), and have been looking for useful things to use it for (I don‘t have much of a plan for it, at the moment). Reading [this](https://www.dzombak.com/blog/2023/12/Considerations-for-a-long-running-Raspberry-Pi.html) led me to [this](https://github.com/cdzombak/nut_influx_connector), which led me to the [Network UPS Tools](https://networkupstools.org/) project.

Home Assistant has [native NUT support](https://www.home-assistant.io/integrations/nut/), so I thought it may be useful to get HA monitoring a UPS or two.

After watching a couple [helpful](https://www.youtube.com/watch?v=OM4bY6ViZrg) [videos](https://www.youtube.com/watch?v=vyBP7wpN72c&t=0s), I still couldn‘t decide exactly which parts of NUT needed to be running where. There are several different layers and components to NUT (driver, server, client) and modes (primary, secondary, netserver, standalone, etc). Determining which modes to use for which components for the setup I was going for wasn‘t obvious from the tutorials I was watching and reading.

Here‘s what I figured out, but please know that this could be very wrong, since I only learned of NUT this morning. This assumes that you have a UPS somewhere connected to a computer (probably via USB). This computer is running NUT **only** to provide UPS status and statistics to other computers. It is not doing any real monitoring itself – that‘s what Home Assistant is for.

Configuration of the UPS driver is required. This is done in [`/etc/nut/ups.conf`](https://networkupstools.org/docs/man/ups.conf.html), and looks something like:

```ini
pollinterval = 1

[mygreatups]
    driver = usbhid-ups
    port = auto
    desc = "The power of 1,000 suns"
    vendorid = 0123
    productid = 0987
```

This configuration basically registers a specific, attached UPS to the rest of the NUT system. If you don‘t do this, NUT won‘t communicate with the UPS.

A NUT server is what deals with data coming into the computer from a UPS via the driver. This is true no matter what ends up consuming that data. If you want to do anything with data coming out of a UPS, the NUT server (called `upsd`) must be involved. Therefore, the configuration in [`/etc/nut/upsd.conf`](https://networkupstools.org/docs/man/upsd.conf.html) is required.

```ini
LISTEN 0.0.0.0 3493
```

For my setup, where some external, Home Assistant computer is going to be collecting data about this UPS, the server is configured to `LISTEN` to requests from all IPs on the default NUT port. (If my Home Assistant had a static IP, this could be configured to `LISTEN` only to that, I suppose.)

There‘s another config file called [`/etc/nut/nut.conf`](https://networkupstools.org/docs/man/nut.conf.html), which seems to have overarching control of various NUT layers, and can influence both the server and client. I believe this file will exist no matter what sort of setup you‘re designing, and will always declare a `MODE`.

```ini
MODE=netserver
```

`netserver` is the mode that allows other computers to talk to the server, based on the `LISTEN` directive declared in `upsd.conf`.

Once the NUT server is running, the NUT integration in Home Assistant will be able to discover it. If you try to connect, it will ask for credentials. These are defined in `/etc/nut/uspd.users`.

```ini
[observer]
        password = some_passwd
        upsmon secondary
```

The username `observer` seems to be a defacto standard for monitoring purposes, at least in the docs, but it could be anything. The `upsmon` option, as best I can tell, describes the relationship between the UPS and the computer runnning `upsmon`. I believe that in my case, the Home Assistant is running `upsmon`, so `secondary` is the correct choice, since Home Assistant can‘t locally manage the UPS. I could be misunderstanding where `upsmon` runs or what it does, but from everything I‘ve seen, `upsmon` only comes into play if `upsmon.conf` has been configured. Since I haven‘t configured that on the computer attached to the UPS, I don‘t think `upsmon` is relevant to this user, and so I‘ve called is `secondary`. 🤷‍♂️.

After configuring those four files, and restarting, everything worked as expected. I was able to add the NUT device in Home Assistant using the `observer` username and password. I think that basically starts running a NUT client on the Home Assistant computer, and handles any configuration needed for you. Home Assistant then has access to the device and sensors like status, charge, load, etc. Which is exactly what I wanted.

My computer-with-a-UPS-attached was a Raspberry Pi, so I installed NUT through `apt`. I only installed the `nut` package; there are other related packages, but they were not necessary for this setup.

So I believe what this yields is: a computer running a NUT-provided UPS driver and a NUT server that makes the data from that UPS available via an API. The API can be accessed using the credentials that are defined. Home Assistant and the NUT integration run a NUT client that pulls data from the server‘s API.

The thing you‘ll see mentioned in lots of videos or guides is the creation of monitors within `upsmon.conf`. Note that for this setup, I did not have to create any monitors by hand or do anything with `upsmon.conf`. I assume that is happening somewhere inside Home Assistant, but really couldn‘t say for sure. I also don't know if there‘s a distinction between `upsmon` and a NUT client. Making this all do something is a project for another day. Today was just about seeing numbers.
