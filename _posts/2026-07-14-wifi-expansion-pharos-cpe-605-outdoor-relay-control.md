---
layout: post
title: "Extending Wi-Fi to the second property with two TP-Link Pharos CPE605 units"
slug: wifi-expansion-pharos-cpe-605-outdoor-relay-control
date: 2026-07-14 09:00:00 +0200
categories: [Automation, Networking, IoT]
tags: [WiFi, TP-Link, Pharos, CPE605, ESP8266, MQTT, Home Assistant, Relay]
summary: "How I planned a 40-meter outdoor Wi-Fi bridge using two TP-Link Pharos CPE605 devices to control lights and pumps with an ESP8266 relay module."
---

## Why the expansion is needed

I have a second property about **40 meters away from the house**, and I wanted to extend my network there in order to control:

- outside lights on the second property,
- the goat barn light,
- water pumps,
- and other outdoor automation tasks.

This is not a massive distance, but it is far enough that regular indoor Wi-Fi does not reach reliably through walls and trees.
For this kind of project I wanted a dedicated outdoor bridge that can carry the control traffic cleanly and avoid flaky wireless coverage.

## The plan: two TP-Link Pharos CPE605 units

The plan is to use two **TP-Link Pharos CPE605** units as a point-to-point bridge.
One unit lives near the home base and acts as the access side, while the second unit is installed on the second property and connects the remote relay controller back to the main network.

### Why the CPE605?

- Outdoor-ready hardware with a directional antenna
- 5 GHz band for less interference and better throughput
- Built-in PoE for easy outdoor power
- Designed for point-to-point / point-to-multipoint links

That makes it a good choice for a 40-meter outdoor extension where I want stable remote control rather than a weak range extender.

## Rough wiring and topology

The link itself is simple:

```text
Home base (router / switch)
        |
        | Ethernet + PoE
        |
  [CPE605 #1 - AP/Bridge mode]
        |  5 GHz wireless link
        |
  [CPE605 #2 - Station mode]
        |
        | Ethernet
        |
  [ESP8266 relay controller enclosure]
        |
        +- Relay outputs -> outside lights
        +- Relay outputs -> goat barn light
        +- Relay outputs -> water pump(s)
```

At the home end, the first CPE605 is connected to my local network and gets power through PoE.
At the remote property, the second CPE605 is mounted outdoors and feeds a small wired enclosure with the ESP8266 relay module.

## What the remote controller does

The actual automation device on the second property is based on an **ESP8266** running the project from:

https://github.com/eriksleziona/esp8266_relay_module

That board handles the relay driving and the MQTT communication.
From Home Assistant or any MQTT client, I can now toggle the relays and control:

- the barn light as a normal switch,
- the outside light as a normal switch,
- the water pump with a keyed command,
- and potentially a momentary trigger if I want a pulse output.

## Summary of the ESP8266 relay software

The repository is a MicroPython-based relay controller for a 4-channel relay board.
It includes:

- Wi-Fi connection and reconnection logic,
- MQTT control,
- Home Assistant MQTT discovery support,
- retained relay state publishing,
- availability reporting using MQTT last will.

### Default pin mapping

By default, the code uses the standard ESP8266 relay pins:

- Relay 1 -> GPIO5 (`D1`)
- Relay 2 -> GPIO4 (`D2`)
- Relay 3 -> GPIO14 (`D5`)
- Relay 4 -> GPIO12 (`D6`)

The script assumes an active LOW relay board, which matches the common cheap 4-channel relay modules.

### Basic configuration example

Here is the configuration pattern used in `main.py`:

```python
WIFI_SSID = "YOUR_WIFI_SSID"
WIFI_PASSWORD = "YOUR_WIFI_PASSWORD"

MQTT_BROKER = "192.168.1.100"
MQTT_PORT = 1883
MQTT_USER = "mqtt_user"
MQTT_PASSWORD = "mqtt_pass"

DEVICE_NAME = "ESP8266 4CH Relay"
DEVICE_PREFIX = "esp8266-relay4"
BASE_TOPIC = "home/%s" % DEVICE_PREFIX
```

Use a unique `DEVICE_PREFIX` for each board so that MQTT topics and Home Assistant entities stay separate.

### Relay mode examples

A normal retained relay looks like this:

```python
{
  "channel": 2,
  "name": "Pump Relay",
  "pin": 4,
  "mode": "switch",
  "startup": "OFF",
  "icon": "mdi:pump",
}
```

A momentary trigger can be configured as a button relay:

```python
{
  "channel": 1,
  "name": "Gate Trigger",
  "pin": 5,
  "mode": "button",
  "pulse_ms": 700,
  "icon": "mdi:gate",
}
```

This gives the project flexibility if I ever want a short pulse output instead of a latched on/off relay.

## How Home Assistant sees it

The code publishes MQTT discovery to Home Assistant using topics like:

- `homeassistant/switch/.../config` for switch relays
- `homeassistant/button/.../config` for button relays

After the ESP8266 connects to Wi-Fi and MQTT, Home Assistant should discover the relays automatically.
That means the second property can be controlled without a manual YAML setup for every relay.

## What comes next

The next steps are:

1. install the CPE605 units and align them for a clean 5 GHz link,
2. run Ethernet from the remote CPE605 to the ESP8266 enclosure,
3. finalize the relay configuration for the barn light, outside lamp, and pumps,
4. verify MQTT and HA discovery over the new link,
5. lock the remote enclosure and test under real outdoor conditions.

## Why this setup makes sense

For this kind of automation, a short dedicated outdoor bridge is usually more reliable than trying to stretch indoor Wi-Fi.
Using the TP-Link Pharos CPE605 pair gives me a stable wireless backbone, and the ESP8266 relay module provides a small, inexpensive remote I/O board that can be controlled from the main house.

Once the link is live, the second property will behave like a natural extension of the home network, and the relay board can manage lights and pumps safely from Home Assistant.
