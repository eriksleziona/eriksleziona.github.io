---
layout: post
title: "SSH Hardening: Replacing Passwords with Cryptographic Keys"
date: 2026-04-10
categories: [Security, Linux]
tags: [SSH, Raspberry Pi, Debian, MQTT, Home Assistant]
summary: "How I hardened SSH access on a Raspberry Pi Zero by switching from passwords to key-based authentication, disabling root login, and fixing a small port configuration mistake."
---

## Context

One of the devices in my home lab is a **Raspberry Pi Zero** used to work with an **eBUS controller** and forward data over **MQTT** to my **Home Assistant** server.

The device runs a **Debian Bookworm Lite** setup, and because it is part of my local infrastructure, I wanted to improve its SSH security before leaving it in regular use.

The related project code is available on my [GitHub profile](https://github.com/{{ site.github_username }}).

## The Goal

The main objective was simple:

1. Stop relying on password-based SSH logins.
2. Use cryptographic keys instead.
3. iDisable root logn.
4. Move SSH away from the default port `22`.

## Step 1: Generating and Deploying the SSH Key

On my local machine, I generated an SSH key pair with `ssh-keygen`.

After that, I used `ssh-copy-id` to send the public key to the Raspberry Pi over its local IP address. Once the key was copied successfully, I logged into the device and confirmed that key-based authentication was working.

## Step 2: Hardening the SSH Configuration

After logging into the Raspberry Pi, I edited the SSH configuration files in `/etc` using `nano`.

The important security changes were:

- `PasswordAuthentication no`
- `PermitRootLogin no`
- `Port 3050`

I also reviewed both `ssh_config` and `sshd_config`, but the server-side behavior that matters for incoming SSH connections is controlled by `sshd_config`.

## Step 3: Restarting the Service

Once the configuration changes were saved, I restarted the SSH service:

```bash
sudo systemctl restart ssh
```

To verify whether the new port had actually been applied, I checked the effective SSH daemon configuration with:

```bash
sshd -T | grep port
```

## The Problem

Even after restarting the service, the output still showed port `22`.

At first, I double-checked whether I had saved the file correctly, and I had. The configuration file was definitely edited.

That is when I realized I had made a classic beginner mistake: I changed the port value, but I had **not uncommented the `Port` directive**. So even though `3050` was written in the file, the active configuration was still using the default port `22`.

## The Fix

After uncommenting the `Port` line properly, I restarted the SSH service again:

```bash
sudo systemctl restart ssh
```

Then I repeated the validation:

```bash
sshd -T | grep port
```

This time, the active port was correctly shown as `3050`.

## Outcome

After the correction:

- SSH login worked using my device key instead of a password.
- Password-based authentication was disabled.
- Root login was disabled.
- The SSH service was listening on port `3050` instead of `22`.

From that point on, I could log into the Raspberry Pi securely with key-based authentication only.

## Key Takeaway

This was a good reminder that hardening is not just about changing settings. It is also about **validating what the service is actually using**, not what I think I wrote in the config file.

Sometimes the difference between a secure configuration and the default one is just a missing uncommented line.
