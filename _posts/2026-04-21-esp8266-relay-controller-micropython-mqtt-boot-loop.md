---
layout: post
title: "ESP8266 Relay Controller: MicroPython, MQTT, and One Very Angry Boot Loop"
slug: esp8266-relay-controller-micropython-mqtt-boot-loop
date: 2026-04-21 09:20:00 +0200
categories: [Automation, Home Assistant]
tags: [ESP8266, MQTT, MicroPython, Home Assistant, IoT]
summary: "How I built a 4-channel ESP8266 relay controller with MicroPython and MQTT autodiscovery, hit a stubborn boot loop, and brought the board back to life with esptool."
---

## Context

Today I wanted to do something a little different.
Instead of hardening a server or chasing logs, the plan was to build a small relay system using an **ESP8266** and a **4-channel relay module**.

The idea was to control the relays from **Home Assistant** using **MicroPython** and **MQTT autodiscovery**.

Because apparently pressing a physical switch with my hand was starting to feel a little too manual.

## Hardware Setup

I started with the wiring between the NodeMCU-style ESP8266 board and the relay module.
The relay board itself was powered from the controller's `5V` pin, and the control lines were mapped like this:

```python
# NodeMCU pin mapping:
# D1 = GPIO5, D2 = GPIO4, D5 = GPIO14, D6 = GPIO12

# mode:
#   "switch" -> stays ON/OFF until commanded
#   "button" -> pulses for pulse_ms and returns to OFF
```

This gave me enough flexibility to use a relay either as a normal switch or as a short pulse trigger, depending on what I want to control later.

<figure>
  <img src="{{ '/assets/images/esp8266-esptool-wiring.png' | relative_url }}" alt="ESP8266 controller mounted inside an enclosure and wired to a 4-channel relay board." />
  <figcaption>The real wiring photo: ESP8266 on the left, relay board on the right, and a surprisingly decent amount of order for an electronics box.</figcaption>
</figure>

## Why MicroPython and MQTT Instead of ESPHome?

To be fair, **ESPHome** would probably have been the faster and easier route.
It is popular for a reason.

But this time I wanted to experiment, write the logic more directly, and avoid depending too much on a higher abstraction layer.
Part of the fun here was understanding how the device actually behaves.

I also wanted to avoid a situation where one future update arrives with the energy of "good news, your setup is different now."

So I built it around **MicroPython** and **MQTT autodiscovery** instead.

The code for the project is here:
[ESP8266 relay module repository](https://github.com/eriksleziona/esp8266_relay_module)

## Software Side

After wiring the hardware, I moved to the MicroPython script itself:

- connect the ESP8266 to Wi-Fi
- connect it to MQTT
- publish autodiscovery topics for Home Assistant
- expose four relays that can be controlled from the dashboard

I already have some basic Python knowledge, and this time I also used **OpenAI Codex** to help me work through the programming side.
That turned out to be a very good combination: I could stay focused on the logic and the hardware instead of getting stuck on every small implementation detail.

## When Thonny Decided to Add Drama

Then the project reached the phase that every small hardware build seems to include:
the part where the tool that is supposed to help suddenly becomes part of the problem.

I used **Thonny IDE** to copy the code to the ESP8266, but I could not save the file correctly on the device.
I flashed the board and installed firmware, but after that the ESP got stuck in a **boot loop** and the blue LED started blinking like it was trying to send an SOS signal to orbit.

Even better, Thonny itself became unresponsive, so the whole "friendly IDE workflow" was no longer especially friendly.

## Terminal Rescue With esptool

At that point, the fix was to stop negotiating with the IDE and go straight to the terminal.

I downloaded the firmware from the official MicroPython website and flashed the board using `esptool`.
The exact firmware filename depends on the version you download, but the process looked like this:

```powershell
esptool --port COM3 erase-flash
esptool --port COM3 write-flash 0x00000 <firmware-file>.bin
```

Once the board was flashed cleanly this way, the boot loop disappeared and the device started behaving normally again.

<figure>
  <img src="{{ '/assets/images/esp8266-esptool-flash.png' | relative_url }}" alt="Terminal output showing the ESP8266 flash memory being erased successfully with esptool." />
  <figcaption>Sometimes the most advanced troubleshooting strategy is simply to open a terminal and become difficult in return.</figcaption>
</figure>

After that I was finally able to save the file as `main.py`, reboot the board, and continue without the LED performing its own panic routine.

## Result in Home Assistant

With the firmware fixed and the script in place, the relay controller appeared correctly in **Home Assistant** through **MQTT autodiscovery**.

That is always the moment when a project stops feeling like a box full of wires and starts feeling like an actual system.

<figure>
  <img src="{{ '/assets/images/mqtt_data.png' | relative_url }}" alt="Home Assistant dashboard showing MQTT relay controls and device data for the ESP8266 relay controller." />
  <figcaption>The actual Home Assistant view from the repo: relay controls, MQTT data, and the small but satisfying feeling that the whole thing is finally alive.</figcaption>
</figure>

## Outcome

- ESP8266 and 4-channel relay hardware wired correctly
- relay board powered from the controller's `5V` pin
- MicroPython used for custom device logic
- MQTT autodiscovery used for Home Assistant integration
- Thonny and firmware issues worked around by reflashing with `esptool`
- final script saved as `main.py` and the device started working correctly

## Key Takeaway

Sometimes the problem is not your code.
Sometimes it is the firmware, the flashing workflow, or one small step that the board has decided to take personally.

And sometimes the most professional move is to close the GUI, open a terminal, and have a calm but very direct conversation with the hardware.
