---
layout: post
title: "Cron, Not Crown: A Weekly Linux Update Check That Remembers for Me"
date: 2026-05-04 09:30:00 +0200
categories: [Linux, Automation]
tags: [Bash, Cron, Updates, Scripting, Package Management]
summary: "A small Bash script that checks for package updates, writes the results to a log, and lets cron remember the weekly check so I do not have to."
---

## Why I Wanted This

Sometimes the most dangerous sentence in Linux administration is:

> "I will remember to check updates later."

I usually will not.

So the goal here was simple:
write a small Bash script that checks for package updates, save the result to a log file, and let `cron` run it once a week.

Also yes, it is **cron**, not **crown**.
This task needs automation, not a coronation.

## Step 1: Create the Script

Create the file:

```bash
nano ~/check_updates.sh
```

Then paste in this script:

```bash
#!/usr/bin/env bash

PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOGFILE="$SCRIPT_DIR/update_check.log"
DATE="$(date '+%Y-%m-%d %H:%M:%S')"

{
  echo "[$DATE] Checking for updates..."

  if command -v apt >/dev/null 2>&1; then
    if [ "$(id -u)" -ne 0 ]; then
      echo "apt update usually needs root privileges."
      echo "Run this script with sudo or schedule it from root's crontab."
    else
      apt update -qq
      apt list --upgradable 2>/dev/null
    fi
  elif command -v dnf >/dev/null 2>&1; then
    dnf check-update
  elif command -v yum >/dev/null 2>&1; then
    yum check-update
  elif command -v pacman >/dev/null 2>&1; then
    pacman -Qu
  else
    echo "No supported package manager found."
  fi

  echo "----------------------------------------"
} >> "$LOGFILE" 2>&1
```

Make it executable:

```bash
chmod +x ~/check_updates.sh
```

## Step 2: Schedule It Weekly With Cron

Edit your crontab:

```bash
crontab -e
```

Add this line to run the script every Sunday at `03:00`:

```cron
0 3 * * 0 /home/YOUR_USERNAME/check_updates.sh
```

Replace `YOUR_USERNAME` with your real username.

If you are using **Debian** or **Ubuntu** and you want the script to run `apt update`, remember that `apt` usually needs elevated privileges.
In that case, using `sudo crontab -e` is often the cleaner option.

## Why I Like This Approach

This setup is not flashy, but that is exactly why I like it:

- one small script
- one log file
- one cron entry
- zero need to trust my future self to remember a weekly maintenance task

That is a very fair trade.

## Optional Improvements

There are a few nice upgrades you can add later:

- If mail is configured, cron can send job output as an email alert.
- On Debian or Ubuntu, `apt list --upgradable` is often enough once package metadata is already fresh.
- If you want actual automatic patching, that is a different setup such as `unattended-upgrades` or `dnf-automatic`.

## Final Thought

This is one of those tiny Linux admin tasks that will never look heroic on a CV.

But good system administration is often exactly that:
small boring things, done consistently, before they become large exciting problems.

And honestly, I would rather let `cron` remember them than wait for my memory to start negotiating.
