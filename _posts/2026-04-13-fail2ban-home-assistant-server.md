---
layout: post
title: "Fail2Ban: Hiring an Automatic Bouncer for My Home Assistant Server"
date: 2026-04-13 19:15:00 +0200
categories: [Security, Linux]
tags: [Fail2Ban, Debian, Home Assistant, UFW, SSH]
summary: "How I set up Fail2Ban on my Debian Home Assistant server, why I chose it over manual iptables blocking, and how I verified that the sshd jail was ready to start throwing out brute-force guests."
---

## Context

After setting up **UFW** on the Debian server that hosts my **Home Assistant** environment, the next logical step was to add another security layer: **Fail2Ban**.

If UFW is the locked front door, then Fail2Ban is the automatic guard standing behind it, watching the logs and asking suspicious visitors why they keep trying the wrong password.

## Step 1: Update the System

Before installing anything new, I updated the server so it was running the latest available packages:

```bash
sudo apt update && sudo apt upgrade
```

## Step 2: Install Fail2Ban

Then I installed the package:

```bash
sudo apt install fail2ban
```

Fail2Ban is a Python-based intrusion prevention tool. In practice, it watches log files for repeated failed authentication attempts and then blocks the offending IP address automatically for a defined period of time.

That makes it especially useful for protecting services like **SSH**, where brute-force attempts are common and attackers are usually far more patient than administrators.

## Why Not Just Use iptables Manually?

Technically, I could block suspicious IP addresses manually with `iptables`, for example:

```bash
sudo iptables -A INPUT -s 1.2.3.4 -j DROP
```

That means:

- `-A INPUT` adds a rule for incoming traffic.
- `-s 1.2.3.4` targets that specific source IP.
- `-j DROP` silently drops the traffic.

That works, but it has two obvious disadvantages:

1. It is manual.
2. It does not stay persistent after a reboot unless the rules are saved properly.

To make `iptables` rules persistent, I would also need:

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

So yes, manual blocking is possible.
It is just not very elegant when the internet insists on being rude at scale.

## Step 3: Create a Safe Local Configuration

Instead of editing the default configuration directly, I created a local copy:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

That way, my custom settings stay separate from the default file.

## Step 4: Configure the Global Defaults

Inside `/etc/fail2ban/jail.local`, I adjusted the main settings in the `[DEFAULT]` section:

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
ignoreip = 127.0.0.1/8 YOUR_IP
```

These values define the basic behavior:

- `bantime` is how long an IP stays banned.
- `findtime` is the time window Fail2Ban uses to count failed attempts.
- `maxretry` is how many failures are allowed before the ban happens.
- `ignoreip` is critical, because it prevents me from accidentally banning myself from my own server, which would be a very efficient but deeply annoying security policy.

## Step 5: Enable SSH Protection

The most important jail for my setup was `sshd`, so I enabled it in `jail.local`:

```ini
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
```

If SSH is running on a custom port, that `port` value should be changed to match it.

## Step 6: Start and Enable the Service

After the configuration was in place, I enabled and restarted the service:

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

Then I checked the status:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

<figure>
  <img src="{{ '/assets/images/fail2ban-status.png' | relative_url }}" alt="Terminal output showing fail2ban-client status with the sshd jail enabled and currently no banned IPs." />
  <figcaption>Fail2Ban up and running, with the SSH jail active and ready to play the role of automatic bouncer.</figcaption>
</figure>

## Outcome

At the end of the setup:

- Fail2Ban was installed on the Debian server.
- A local `jail.local` configuration was created instead of editing the default file.
- The global banning rules were customized.
- SSH protection was enabled.
- The service was active and responding correctly to status checks.

This gave the server a much more practical defensive layer than manually chasing suspicious IPs one by one.

## Next Step

The next step will be the fun part: testing Fail2Ban with a controlled attack against my own server to confirm that the bans trigger exactly as expected.

In security work, trust is good.
Verification is better.
