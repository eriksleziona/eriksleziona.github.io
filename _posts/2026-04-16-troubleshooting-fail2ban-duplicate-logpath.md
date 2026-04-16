---
layout: post
title: "Troubleshooting Fail2Ban: When the Bouncer Trips Over His Own Config"
date: 2026-04-16 09:15:00 +0200
categories: [Troubleshooting, Linux]
tags: [Fail2Ban, Debian, Cockpit, SSH, Logs]
summary: "How I tracked down a Fail2Ban startup failure on my Debian server, followed the error into jail.local, and fixed a duplicate logpath entry that had broken the service."
---

## Context

Today, during a routine health check of the server, I noticed that the **Fail2Ban** service had failed to start.

That was slightly ironic.
The security tool meant to defend the server had quietly fallen over like a guard who tripped over his own shoelaces before the shift even started.

## Step 1: Confirm the Service Failure

The first step was to log into **Cockpit**, open the terminal, and check the service state.

Even before going deep into the terminal, Cockpit was already being very direct about the problem:

<figure>
  <img src="{{ '/assets/images/fail2ban-cockpit-service-failed.png' | relative_url }}" alt="Cockpit service list showing the Fail2Ban service failed to start." />
  <figcaption>Not exactly the reassuring green status you hope to see from a security service.</figcaption>
</figure>

Then I checked it from the command line:

```bash
systemctl status fail2ban
```

<figure>
  <img src="{{ '/assets/images/fail2ban-systemctl-failed.png' | relative_url }}" alt="Terminal output from systemctl status fail2ban showing the service in a failed state with an exception exit code." />
  <figcaption>The service was definitely down, and a manual restart was not going to magically fix bad configuration.</figcaption>
</figure>

The result was clear: the service had failed, and manually restarting it did not solve the problem.

## Step 2: Stop Guessing and Check the Logs

At that point, it was time to stop negotiating with the problem and go to the logs.

That is where the useful clue showed up:

<figure>
  <img src="{{ '/assets/images/fail2ban-log-error.png' | relative_url }}" alt="Log entry from fail2ban-server reporting that the option logpath in the sshd section already exists in jail.local." />
  <figcaption>The logs were kind enough to point directly at the guilty line, which is more than can be said for many errors in life.</figcaption>
</figure>

The key part of the message was:

```text
option 'logpath' in section 'sshd' already exists
```

That told me this was not a package issue or a broken service binary.
This was a configuration problem, and most likely one I had introduced myself.

## Step 3: Inspect `jail.local`

After reading the log message, I opened the `jail.local` file that I had previously created to configure Fail2Ban.

There it was:

<figure>
  <img src="{{ '/assets/images/fail2ban-jail-local-logpath-error.png' | relative_url }}" alt="Snippet of the sshd section in jail.local showing two logpath entries, including an incorrect duplicated line." />
  <figcaption>The classic moment when the config file politely reveals that yes, the problem is absolutely you.</figcaption>
</figure>

Inside the `[sshd]` section, I had duplicated the `logpath` setting:

```ini
logpath = %(sshd_log)s
logpath = logpath = /var/log/auth.log
```

That second line was clearly wrong.
It duplicated the parameter name and created a configuration conflict that prevented Fail2Ban from starting at all.

## Step 4: Clean the File and Restart the Service

Once I found the issue, the next step was simple:

1. Clean up the `jail.local` file.
2. Remove the broken duplicate `logpath` entry.
3. Review the file for any other mistakes.
4. Restart the service.

After fixing the configuration, I restarted Fail2Ban again, and this time it started normally.

## Outcome

At the end of the troubleshooting session:

- The service failure was confirmed from both Cockpit and the terminal.
- The logs pointed directly to a configuration error in `jail.local`.
- The root cause turned out to be a duplicated `logpath` parameter in the `[sshd]` section.
- After cleaning the file and restarting the service, Fail2Ban worked again.

## Key Takeaway

Sometimes troubleshooting looks dramatic at first, but the actual problem is just one broken line in a file you edited yourself last week.

Fail2Ban was not broken.
It was simply refusing to work with a configuration that was arguing with itself.
