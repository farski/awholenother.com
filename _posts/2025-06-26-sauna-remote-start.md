---
layout: post
title: Adding remote starter to a Costco infrared sauna
date: 2025-06-26 11:46 -0400
reading_time: 12 minutes
tags:
  - sauna
  - DIY
  - smarthome
  - electronics
  - microcontrollre
---

I have [this infrared sauna](https://dynamicsaunasdirect.com/products/santiago-2-person-low-emf-far-infrared-sauna?srsltid=AfmBOoqarQ9SfhPoRprAxfNwZg7lVcThlGRLrvkmtWAcYsGQ73lr46ub). I don't really understand the difference between [Golden Designs Inc](https://goldendesigninc.com/collections/dynamic-saunas/products/santiago-2-person-low-emf-far-infrared-sauna) and [Dynamic Saunas Direct](https://dynamicsaunasdirect.com/products/santiago-2-person-low-emf-far-infrared-sauna?srsltid=AfmBOoqarQ9SfhPoRprAxfNwZg7lVcThlGRLrvkmtWAcYsGQ73lr46ub), but they pretty much seem to be two names for the same company. I bought the sauna through Costco, and they [list the](https://www.costco.com/dynamic-santiago-2-person-full-spectrum-infrared-sauna.product.4000348956.html) brand as "Dynamic Saunas". Regardless of the brand, everyone seems to agree that the model number is `DYN-6209-01`.

In the world of saunas, I'm sure this one is not great. But it was a affordable, and, as a renter, most other saunas were not practical. Even slightly nicer IR saunas generally require 20 A, which would have required wiring work I didn't want to do. But this is a box that gets relatively hot, and that's what I was looking for, so I've been happy with it.

The one feature it's lacked that I've really been wanting in the year or so that I've had it is some way to start the sauna heating up remotely or on a delay. It takes about 30 minutes to get to a reasonable temp (or about 45 minutes to get its max temp), which means if I go for an hour long run in the middle of winter, I would really want the sauna to start heating up 30 minutes after I leave. Or if I go skiiing, it'd be great to start it up when I'm about 30 minutes away on the drive home.

Out of the box, the sauna has no support for any sort of delayed start or smart home functionality. Unlike some traditional saunas, there's no option for this one to start heating as soon as it's given power, so simply putting it on a smart outlet doesn't solve the issue.

The sauna is controlled through a [panel](https://dynamicsaunasdirect.com/products/control-panel-model-dyn-cp-green-sl) that's inside the sauna. When you first plug the sauna into power, it is in an idle state. You then press the **POWER** button on the panel, which activates the rest of the panel and the other features of the suana (interior lights, built-in bluetooth speaker, etc), but it doesn't start powering the heating panels. You have to additionally press the **WORK** button on the panel for things to start heating up.

The control panel PCB has three connections: a 2-wire connector that goes to the speakers, a 2-wire connector coming from the 3.5 mm TRS aux jack, and a 10-wire Micro-Fit 3.0 (2 row) connector. Those 10 wires go to various places.

When I started thinking about adding remote start functionality to the sauna, my first idea was to try to decode whatever signalling was being used to send commands from the control panel to the brain/power supply on the roof that actually controls the heating panels. People on YouTube with oscilloscopes make it look so easy to pull apart UART signals. Eventually I decided that I didn't want to buy an oscilloscope, and even if I had one, that was too much to bite off for this project. My experience with this sort of thing is pretty limited, and hacking the components at that level felt like too much of a risk, and I probably would have ended up with a broken sauna.

So rather than talking to the brain directly, I switched focus to interacting with the control panel. All I really need to be able to do is activate the POWER and WORK buttons, without actually being there to physically press them. My second idea was to connect something like an ESP32's GPIO to the switches' pins on the PCB, and close those circuits to simulate a button press. Again my lack of any real electrical engineering experience put this out of reach. I'm still not entirely sure doing it that way is possible; some sources indicate yes, others no, and others "it depends". That was way too much uncertainty for me, so I went for the safer, third option.

The implementation I ultimately went with was using one of these [ESP8266 dual channel relays](https://www.amazon.com/dp/B0BXDK5GT4). I already had some devices running [ESPHome](https://esphome.io) via [Home Assistant](https://www.home-assistant.io), so I felt comfortable adding another one and dealing with whatever programming or configuration was necessary. This setup allows you to open and close each relay through Home Assistant.

For each of the POWER and WORK buttons on the sauna's control panel, I wired one side of the button to one side of the relay, and vice versa. That would mean closing the relay would be the same as depressing the button on the control panel. Thus, closing and opening the relay quickly would simulate a normal button press.

I found a bunch of disagreeing examples of setting up this particular relay board in HA, and mostly just tried different combinations of settings until I got mine working reliably. I'm not sure where the differences in things like UART baud rate or commands come from, if you compare the examples.

```yaml
esphome:
  name: sauna-relay
  friendly_name: sauna-relay

esp8266:
  board: esp01_1m

logger:
  baud_rate: 0 # need this to free up UART pins

api:
  encryption:
    key: "tktktk"

ota:
  - platform: esphome
    password: "tktktk"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Sauna-Relay Fallback Hotspot"
    password: "tktktk"

captive_portal:

web_server:
  port: 80

uart:
  baud_rate: 115200
  tx_pin: GPIO1
  rx_pin: GPIO3

# The format is the following:
# first byte:  0xA0
# second byte: relay number (first is 0x01, second is 0x02)
# third byte: command (0x00-off, 0x01-on)
# fourth byte: sum of previous bytes

switch:
  - platform: template
    name: "Power Control"
    icon: mdi:power
    optimistic: true
    restore_mode: ALWAYS_OFF
    turn_on_action:
      - uart.write: [0xA0, 0x01, 0x01, 0xA2]
    turn_off_action:
      - uart.write: [0xA0, 0x01, 0x00, 0xA1]
  - platform: template
    name: "Heat Control"
    icon: mdi:heat-wave
    optimistic: true
    restore_mode: ALWAYS_OFF
    turn_on_action:
      - uart.write: [0xA0, 0x02, 0x01, 0xA3]
    turn_off_action:
      - uart.write: [0xA0, 0x02, 0x00, 0xA2]
```

This will result in two switch entities in HA that you can turn on and off in sync with the physical relays.

To make simulating button presses more convenient, I also created a HA script for each button, like this:

```yaml
sequence:
  - type: turn_off
    device_id: aaaaaaaaaa
    entity_id: ffffffffff
    domain: switch
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 500
  - type: turn_on
    device_id: aaaaaaaaaa
    entity_id: ffffffffff
    domain: switch
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 100
  - type: turn_off
    device_id: aaaaaaaaaa
    entity_id: ffffffffff
    domain: switch
alias: Sauna Simulate Power Button Press
description: ""
```

This script will first make sure the relay is open, in case somehow it's been left closed (this step basically never does anything), and then it quickly closes the relay (simulate depressing the button) and opens the relay (simulate releasing the button). So when this script gets run, it should be the same as someone pressing the POWER button on the control panel. Running this script and then the equivalent script for the HEAT button will take the sauna from idle state to heating. Running the power button script again will take it back to idle state.

Other than the little bit of trial and error with the config, this approach basically worked as I'd hoped right away. Anywhere I had access to my Home Assistant (which is anywhere that I have internet on my phone, because I run them both on a Tailscale network), I could turn the sauna on and off, and turn the heat on and off.

With the core functionality sorted out, the remaining bits were getting the relay permanently installed, and making the automation a bit more convenient.

Installation of the relay was the part of this project that took the longest. During development, I was running the relay off it's own 5 V power supply. Ideally I wanted the relay board to run off the same power as rest of the sauna, mainly so that there was still a single cable that powered eveything for the sauna from the wall. The main power supply on top of the sauna has two unused connectors labeled "12 V", but it turns out these aren't provided constant power; they were powered only when the sauna was in the powered on state (i.e., not when idle). I wanted the relay to be able to wake the sauna up from idle, so this unfortunately wasn't the solution.

A bit of poking around revealed that the control panel runs off 12 V coming over that 10-pin connector. My plan was to run the relay board off that, after running it through a basic [step down transformer](https://www.amazon.com/dp/B09X1XJYB6), to get the 5 V that the board needs.

I wanted all the modifications I was making to the sauna to be reversible, so hacking up the cable that was in there wasn't a great option. Instead, I picked up [a kit](https://www.amazon.com/dp/B0CL9MQMLV) to build out Micro-Fit cables and made a new 8-pin cable to insert between the cable running between the control panel and the brain/power supply (only 8 of the 10 cables leaving the control panel end up at the power supply). I could then splice the 12 V line on my new cable, and send it to the transformer to provide power to the relay board. If necessary, that entire assembly can be removed and the sauna will be back to how it came out of the box.

My first attempt at building that cable almost worked, but I guess I found a pair of pins on the power supply connector that usually has 12 V, but also sometimes doesn't and tapping it prevented the heat panels from heating up even though the control panel thought they were on. After some more multimetering, I found the correct pins to use. That got all the hardware into a spot that I felt pretty good about. As long as the sauna was plugged into the wall, the relay board would have power and I would be able to control it through Home Assistant. It was as close to a native-feeling solution as I could hope for, with my limited EE skills.

The final thing I did was add some additional helpers, scripts, and a dashboard in HA to support delayed start. In some cases (like driving home from skiing), it's easy to press the button in HA when I'm 30 minutes from home. In other cases (like going for an hour long run), I won't be able to, so what I really want is to do something before I leave for the run that tells HA to heat the sauna after a 30 minute delay.

I'm not going to share exactly how I did that. I don't spend too much time in the guts of HA, so while I can sort of get it to do what I want, I'm very sure I'm not doing it in a smart way. But the end result is that I have a `Timer` entity, and I can set its duration. When the timer is done, an automation runs, which triggers a script that runs both the POWER and HEAT button scripts. Ultimately, though, after more steps than should be necessary, the sauna turns on.

To make this process even easier, I made a single [Shortcut](https://apps.apple.com/us/app/shortcuts/id915249334) which lives in a widget, that prompts to start the sauna now, after 30 minutes, after 60 minutes, or after an arbitrary duration. This means I don't even have to open the HA app to start or schedule the sauna. This was the goal of the project, and even though there probably were better ways to get there, I'm pretty happy with how it came out.

The final installation looks like this:

- Custom 8-pin Micro-Fit cable added between the sauna's power supply and the cable running to the control panel.
- 2 wires (12 V) spliced off that custom cable which go to the 12-to-5 V transformer.
- 2 wires (5 V) from the transformer to the relay board.
- 2 wires from each relay that run from the roof of the sauna, down through the wall to the control panel. I used speaker wire because it's what I had lying around.
- Each of those wires is connected to the pins of the correct button switch assembly on the PCB

_(Sorry for the low quality photos)_

![Sauna roof with relay installation](https://kala.farski.com/awholenother/sauna_6470.jpg)
![Alternate angle of relay installation](https://kala.farski.com/awholenother/sauna_6471.jpg)
![Control panel installation](https://kala.farski.com/awholenother/sauna_6467.jpg)
