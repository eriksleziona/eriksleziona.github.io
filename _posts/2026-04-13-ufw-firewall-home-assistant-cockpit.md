---
layout: post
title: "UFW Firewall: Zero Trust, One Open Port, and One Brief Moment of Panic"
date: 2026-04-13
categories: [Security, Linux]
tags: [UFW, Firewall, Debian, Home Assistant, Cockpit]
summary: "How I configured UFW on a Debian VM hosting Home Assistant, accidentally locked myself out of Cockpit for a moment, and then fixed it by allowing only the port I actually needed."
---

## Context

This time I was working on the Debian Linux server that hosts my **Home Assistant** environment inside a **virtual machine**.

The server also uses **Cockpit**, the open-source web interface for Linux administration. For a beginner-friendly setup, Cockpit is honestly great. It makes spinning up and managing virtual machines much less intimidating, which is very useful when you are still building confidence and trying not to set your infrastructure on fire before lunch.

## The Goal

The goal was to configure **UFW** to follow a more locked-down approach:

1. Deny incoming traffic by default.
2. Allow outgoing traffic by default.
3. Open only the ports that are actually needed.

In other words: less "everything is open because it is convenient," and more "if you want to come in, please state your business."

## Step 1: Installing UFW

The first step was installing the firewall:

```bash
sudo apt install ufw
```

## Step 2: Setting the Default Rules

I started with a deny-by-default policy for incoming traffic:

```bash
sudo ufw default deny incoming
```

Then I allowed all outgoing traffic:

```bash
sudo ufw default allow outgoing
```

That gave me a much cleaner baseline: block unsolicited inbound traffic, but still let the server communicate outward as needed.

## Step 3: Enabling the Firewall

After setting the rules, I enabled UFW:

```bash
sudo ufw enable
```

And that was the exact moment my brain produced the very calm and professional internal message:

> I may have just broken this.

## The Problem

The issue appeared immediately after enabling the firewall. I was no longer able to log in through **Cockpit**.

That made perfect sense in hindsight. I had created a "zero trust" style baseline, but I had not yet allowed the port used by the web interface I actually depended on.

So yes, the firewall was doing its job.
Unfortunately, its job included blocking me.

## The Recovery

Luckily, this was not a remote disaster. The hardware was physically located under my desk, so I could connect directly to the server and repair my own mistake without needing an existential crisis or a rescue USB.

To keep the environment locked down while still allowing management access, I opened only the port used by Cockpit:

```bash
sudo ufw allow <port>
```

If Cockpit is using its default port, that would typically be:

```bash
sudo ufw allow 9090/tcp
```

After that, the web interface became reachable again.

## Outcome

At the end of the process:

- Incoming traffic was denied by default.
- Outgoing traffic was allowed by default.
- UFW was active on the Debian server.
- Cockpit access was restored by allowing only the required port.

This left me with a more secure baseline and a very useful reminder: when hardening a system, always make sure you allow your management path before congratulating yourself.

## Key Takeaway

UFW is simple, but it is also very honest. If you tell it to block everything incoming, it will absolutely do that, including blocking the nice admin who set it up five seconds earlier.

Security-wise, this was still a good move. The mistake was not enabling the firewall. The mistake was enabling it before explicitly allowing the service I still needed.
