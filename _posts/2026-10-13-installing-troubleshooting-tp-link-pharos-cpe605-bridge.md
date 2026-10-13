---
layout: post
title: "Installing and Troubleshooting a TP-Link Pharos CPE605 Wireless Bridge"
slug: installing-troubleshooting-tp-link-pharos-cpe605-bridge
date: 2026-10-13 09:00:00 +0200
categories: [Networking, IoT, Automation]
tags:
  [WiFi, TP-Link, Pharos, CPE605, Wireless Bridge, Ethernet, Troubleshooting]
summary: "Mounting two TP-Link Pharos CPE605 units, configuring an AP-to-Station bridge, and tracking down an overpowered link and a mis-crimped Ethernet cable."
---

## From the plan to the roof

In an [earlier post about extending Wi-Fi to the second property](/automation/networking/iot/2026/07/14/wifi-expansion-pharos-cpe-605-outdoor-relay-control.html), I described why I needed a network connection there for my Home Assistant projects. The second property does not need a separate internet line for this setup: the wireless bridge carries the home network connection across the roughly 40-meter gap.

This time, I finally installed the two **TP-Link Pharos CPE605** units and worked through the setup. The hardware went where it was meant to be: one antenna on top of the building's chimney, with the other at the remote property. I ran and wired the Cat 6 Ethernet cables between the equipment and the network points.

## The RJ45 wiring battle

Getting the RJ45 plugs right took more attempts than I expected. I was using standard connectors, not pass-through plugs, where the wires extend through the front and can be checked before crimping. With those, it is much easier to spot a conductor that is out of order before it is too late.

I followed the **T568B** colour sequence and re-terminated the plugs until the cables tested correctly. It was a good reminder that wiring can look almost right and still cause a frustrating fault later.

## Configuring the first CPE605

I connected my laptop directly to the first CPE605's LAN port. Because the units did not provide DHCP during setup, I changed the laptop's Ethernet adapter from automatic addressing to a manual address on the same subnet:

- IP address: `192.168.0.100`
- Subnet mask: `255.255.255.0`

That gave me access to the Pharos web interface. The Quick Setup wizard made it straightforward to configure the first unit in **Access Point (AP) mode**.

I did forget one important step on my first pass: changing the CPE605's management address to a static address on my home network. I had to make another trip to the unit with the cable in hand to correct it. Lesson learned: set and record the final management IP before leaving the antenna.

## Setting up the second unit

I logged in to the second CPE605 and configured it in **Client/Station mode**, following the Quick Setup steps. I also assigned it a static management address on my home network so I could reach it later without reconnecting a laptop directly.

Once both units were configured, I connected the Ethernet port at the remote end to a second router configured as an access point. The idea was simple: extend the existing home network into the garage. The link appeared to be up, but there was no internet connection at the access point. Time to troubleshoot.

## A link that was too strong

I went back through the settings and climbed the ladder to check the LEDs. They looked fine. In the AP's **Stations** status, I could see the client unit, but traffic was not passing as expected.

While checking the radio settings, I found that the units' country and channel configuration needed attention. I had temporarily enabled test mode while investigating channel availability. That exposed more channel options during troubleshooting, but it is not a mode to leave enabled. The units should be set to the correct country, **Poland**, and operated only on channels and power levels permitted by local regulations.

The alignment tool in PharosOS showed a strong signal, but the two antennas were only about **40 meters apart**. At that distance, the transmit power was unnecessarily high and was contributing to a poor link. I reduced the transmit power in 5 dBm steps and set the channel width to **20 MHz**. With a suitable permitted channel and lower transmit power, the link became stable and the noise level improved.

## The cable that passed its test

The wireless bridge was finally working, so I connected its remote LAN port to the access point router. Still no internet.

After more troubleshooting, I found the last problem: one Ethernet plug had been crimped with a conductor in the wrong position. My tester indicated that all eight conductors had continuity, but that did not mean the cable was wired in the correct order. One wire was connected to the wrong pin.

I re-terminated the cable using the T568B sequence, tested it again, and connected the access point. This time, the remote network came online.

## What I learned

- Set a static management address on each CPE605 during setup, before packing away the laptop and cable.
- A station can appear in the AP's client list even when the link is not passing useful traffic; check the radio settings and alignment as well as the LEDs.
- A very strong signal is not automatically a better signal. For two directional antennas only 40 meters apart, reducing transmit power made a real difference.
- A continuity tester can confirm that conductors are connected without proving that they are in the correct pin order. Check the wiring sequence, not just that all eight lights come on.
- Use the correct country setting and stay within the channels and transmit-power limits allowed where the equipment is installed.

The CPE605 bridge now brings the home network and internet connection to the garage. I can continue working on the Home Assistant automations there, including the lighting projects I have written about before. The setup took a few extra trips up the ladder, but it also turned a frustrating connection problem into a useful lesson in wireless links and Ethernet cabling.
