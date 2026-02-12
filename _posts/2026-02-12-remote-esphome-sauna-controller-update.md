---
layout: post
title: Remote ESPHome Sauna controller update
date: 2026-02-12 11:17 -0500
reading_time: 8 minutes
tags:
  - sauna
  - DIY
  - smarthome
  - electronics
  - microcontroller
---

This is an update to [Adding remote starter to a Costco infrared sauna](/2025/06/26/sauna-remote-start.html).

Since I first setup up the EPSHome-based remote starter for [this sauna](https://dynamicsaunasdirect.com/products/santiago-2-person-low-emf-far-infrared-sauna?srsltid=AfmBOoqarQ9SfhPoRprAxfNwZg7lVcThlGRLrvkmtWAcYsGQ73lr46ub), my understanding of ESPHome configuration has improved a bit, and I’ve made some changes to the controller software.

I’ve removed the [web server component](https://esphome.io/components/web_server/). I had no need for that functionality, and apparently it “will take up _a lot_ of memory and may decrease stability, especially on ESP8266.”

I’ve also removed the [captive portal component](https://esphome.io/components/captive_portal/) and disabled the **Access Point Mode** on the [WiFi component](https://esphome.io/components/wifi/#access-point-mode). These were also unnecesary for my needs; if the device fails to connect to WiFi, I would resolve that in a way that doesn’t rely on access point mode.

This isn’t a change, but I now understand that setting `baud_rate: 0` on the [logger component](https://esphome.io/components/logger/#configuration-variables) will disable logging via UART. The relay board I’m using is based on the ESP8266. From what I’ve gathered, that chip either only includes one pair of UART pins, or perhaps includes more than one set but it’s much easier to use the first set to communicate with the relays. (I’ve seen some conflicting information about how many UARTs are included, and haven’t dug in enough to confirm which is correct.) By disabling UART logging, those pins are available exclusively for interfacing with the relays.

I now have a better explanation for the required UART baud rate of `115200`. The relay board uses a [STC15L101EW](https://www.stcmicro.com/datasheet/STC15F100-en.pdf) microcontroller between the ESP8266 and the relays, and it seems like with the default configuration that the board comes in, that uses a baud rate of `115200`.

In terms of the configured UART pins, those come from the [ESP8266EX](https://documentation.espressif.com/0a-esp8266ex_datasheet_en.pdf). As indicated by the original ESPHome config, UART Rx is `GPIO3` (aka, pin 25; aka, `U0RXD`), and Tx is `GPIO1` (aka, pin 26; aka, `U0TXD`). (Not relevant for the ESPHome configuration, but that ESP8266 is included as part of the [ESP-01](https://academy.cba.mit.edu/classes/networking_communications/ESP8266/esp01.pdf) module that is a daughter board for the complete relay board, and I believe of the eight headers exposed on the ESP-01, UART Rx is pin 4 (aka, `RXD`) and Tx is pin 8 (aka, `TXD`).) You can see in the [ESP8266 docs](https://esphome.io/components/esp8266/#gpio-pin-numbering) for ESPHome that `GPIO1` and `GPIO3` are special pins for UART.

For this particular ESP-01 board, there is [no more specific](https://registry.platformio.org/platforms/platformio/espressif8266/boards?p=1) PlatformIO board ID available, so the generic [`board: esp01_1m`](https://docs.platformio.org/en/latest/boards/espressif8266/esp01_1m.html) configuration is used.

My understanding of the `switch` configuration is also much better. I previously didn’t really understand why in various example configs I would find used different `platform` values, like `template`, `gpio`, or `uart` to accomplish very similar things. Looking at the ESPHome docs for [switch components](https://esphome.io/components/#switch-components), I now see that each of those platforms is an extension of the [core](https://esphome.io/components/switch/) switch functionality.

Despite the fact that this config uses GPIO pins to communicate with the relays, I don’t believe that the [GPIO switch platform](https://esphome.io/components/switch/gpio/) is appropriate here. I think that’s if you are using the voltage on the pin itself to operate the switch (not data being transfered over the pin).

I believe the [UART switch platform](https://esphome.io/components/switch/uart/) would allow for operating the relays on this controller board, but as with many things in Home Assistant and ESPHome, there are many ways to accomplish the same or very similar results.

The [template switch platform](https://esphome.io/components/switch/template/) that is being used seems to be a higher-level abstraction for interacting with switches. It is entirely based on [actions](https://esphome.io/automations/actions/#actions), rather than the more direct hardware configuration of the UART platform. Ultimately, though, the actions do make [UART calls](https://esphome.io/components/uart/) (e.g., `uart.write: [0xA0, 0x02, 0x01, 0xA3]`), so the fundamental result is the same. Most examples seem to use the template platform, so I decided to stick with that rather than convert to the UART platform, even though that probably would have been possible.

In [this code example](https://devices.esphome.io/devices/ESP-01-4-Channel-Relay-LC-Technology/#configuration-with-momentary-switches-push-buttons-with-1-sek-press-time) for a very similar relay board, I saw that momentary switch functionality was being baked directly into the ESPHome configuration using the `on_turn_on` [trigger](https://esphome.io/automations/actions/#triggers). Previously I was using a Home Assistant [Script](https://www.home-assistant.io/integrations/script/) to allow these relays to mimic momentary switches, which never felt like the correct approach. Moving the behavior into the ESPHome config means that any time the switch is turned `ON` in the HA frontend, the relay will close, wait a bit, and then open again without any additonal HA scripting. For this use case, there is never a need to keep a relay closed from HA’s perspective, so this essentially hides that and makes this switch behave like a button, which is how it’s intended to behave.

Moving this sort of functionality into the ESPHome device makes a lot more sense, so I’m glad I ran across that example.

Here’s the updated configuration:

```yaml
esphome: { name: sauna-relay, friendly_name: sauna-relay }
esp8266: { board: esp01_1m }
logger: { baud_rate: 0 } # 0 to disable UART logging; required to use UART for relays
api: { encryption: { key: "tktktk" } }
ota: [{ platform: esphome, password: "tktktk" }]
wifi: { ssid: !secret wifi_ssid, password: !secret wifi_password }
uart: { tx_pin: GPIO1, rx_pin: GPIO3, baud_rate: 115200 } # speed for STC15L101EW

# UART data format:
# first byte:     always 0xA0
# second byte:    relay number (first is 0x01, etc)
# third byte:     command (0x00 = off, 0x01 = on)
# fourth byte:    sum of first three bytes

switch:
  - platform: template
    name: "Power Control"
    id: relay01
    icon: mdi:power
    optimistic: true
    restore_mode: ALWAYS_OFF
    turn_on_action:
      - uart.write: [0xA0, 0x01, 0x01, 0xA2]
    turn_off_action:
      - uart.write: [0xA0, 0x01, 0x00, 0xA1]
    on_turn_on:
      - delay: 100ms
      - switch.turn_off: relay01

  - platform: template
    name: "Heat Control"
    id: relay02
    icon: mdi:heat-wave
    optimistic: true
    restore_mode: ALWAYS_OFF
    turn_on_action:
      - uart.write: [0xA0, 0x02, 0x01, 0xA3]
    turn_off_action:
      - uart.write: [0xA0, 0x02, 0x00, 0xA2]
    on_turn_on:
      - delay: 100ms
      - switch.turn_off: relay02

```

### Next steps

I would still like to add some level of monitoring of the sauna’s state to this device. Between indicator LEDs on the control panel and some outputs on the brain/power supply, there are things to use as signals to determine if the power or heat are currently on. This would make it possible to confirm that the sauna has started heating after triggering it remotely, but would also mean that the HA controls could behave based on the current state, rather than blindly mimicing button presses. Right now, it’s not hard to turn the sauna on twice, which ends up actually turning the sauna off. Once the controller knows about the current state, it can do a better job ensuring the requested state is being met.

The ESP8266EX does provide plenty of additional GPIO pins to handle this, but the ESP-01 board doesn’t expose the ones that are best suited for this sort of thing as headers. The ESP-01 only exposes `GPIO0` and `GPIO2`, which can be used for this, but also have special purposes during the boot process. So I need to do a bit more research before I’ll feel comfortable making this enhancement.
