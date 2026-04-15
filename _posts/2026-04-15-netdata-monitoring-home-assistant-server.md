---
layout: post
title: "Netdata: Because a Server Should Not Suffer in Silence"
date: 2026-04-15 09:30:00 +0200
categories: [Monitoring, Linux]
tags: [Netdata, Monitoring, Debian, Home Assistant, UFW]
summary: "How I added Netdata monitoring to my Home Assistant server, picked the easy road on purpose, and then spent a few minutes troubleshooting the very firewall rules I had proudly configured earlier."
---

## Context

At some point, every home server reaches the same stage in life:

it stops being enough that the system is *probably fine*.
Now I want proof.

This time, the next step on the Debian server hosting my **Home Assistant** environment was to implement a proper monitoring tool.

There are plenty of strong open-source options out there:

- **Netdata** for a very quick setup and a polished UI
- **Zabbix** for more traditional infrastructure monitoring
- **Elastic Stack** for logs, search, and metrics-heavy workflows

And of course there is the classic observability stack:

- **Prometheus** for metrics collection
- **Grafana** for dashboards
- **Loki** for logs
- **Tempo** for traces

That stack is powerful, but for this server I wanted something much simpler and faster to deploy.

So I chose **Netdata**.

Why?
Because sometimes the best monitoring decision is the one that gets deployed today instead of being "architected beautifully" for the next three weekends.

## Step 1: Install the Required Tooling

Netdata documentation recommends using its installer script instead of relying on a plain package install, because the distro package may be outdated.

So before anything else, I installed `curl`:

```bash
sudo apt install curl
```

## Step 2: First Attempt, First Confusion

My first try looked like this:

```bash
curl -L https://my-netdata.io/kickstart.sh
```

That did not install Netdata.

Technically, it did exactly what I asked: it downloaded the script output to the terminal.
Unfortunately, "printing a script dramatically on screen" is not the same thing as running it.

So I switched to the more useful version:

```bash
curl -L -o kickstart.sh https://my-netdata.io/kickstart.sh
bash kickstart.sh
```

After that, the installation completed successfully.

## Step 3: Where Netdata Should Be

Once installed, Netdata should be reachable on its default port:

```text
http://<server-ip>:19999
```

Simple.
Beautiful.
Almost suspiciously simple.

## The Troubleshooting Begins

When I tried to access the dashboard, it did not load.

So the troubleshooting started.

The first check was whether the service was actually running:

```bash
sudo systemctl status netdata
```

<figure>
  <img src="{{ '/assets/images/netdata-service-status.png' | relative_url }}" alt="Terminal output showing the netdata service in an active running state on the Debian server." />
  <figcaption>Good news: Netdata was alive. Bad news: the browser still disagreed.</figcaption>
</figure>

The service looked healthy, so the next question was whether the server was listening on port `19999`:

```bash
sudo ss -tulnp | grep 19999
```

<figure>
  <img src="{{ '/assets/images/netdata-port-19999.png' | relative_url }}" alt="Terminal output showing that Netdata is listening on TCP port 19999 on both IPv4 and IPv6." />
  <figcaption>The port was listening exactly where it should have been. Which meant the problem had moved elsewhere, just to stay interesting.</figcaption>
</figure>

## The Real Problem

After confirming that the service was running and the port was open locally, I took two minutes to breathe, stop blaming Netdata, and actually think.

Then it clicked.

Earlier, I had configured **UFW** to deny incoming traffic by default.
At that point, I had only allowed port `9090` for **Cockpit**.

So of course Netdata was unreachable from the browser.
The server was not broken.
The firewall was simply doing exactly what I had told it to do, which is always a slightly humbling moment.

To confirm that, I checked the firewall rules:

```bash
sudo ufw status
```

And yes, port `19999` was missing.

## The Fix

To allow Netdata traffic, I opened the required port:

```bash
sudo ufw allow 19999
```

After that, the dashboard loaded correctly.

And finally, I could see the system metrics in the browser instead of just seeing my own earlier decisions coming back to haunt me.

<figure>
  <img src="{{ '/assets/images/netdata-dashboard-overview.png' | relative_url }}" alt="Netdata dashboard showing a dark monitoring interface with system metrics, CPU charts, and resource overview panels." />
  <figcaption>At last: graphs, panels, metrics, and the deeply reassuring feeling that the server now had something to say.</figcaption>
</figure>

## Outcome

At the end of the setup:

- Netdata was installed successfully using the recommended script-based method.
- The service was confirmed to be running.
- Port `19999` was confirmed to be listening.
- The real issue turned out to be the firewall, not the application.
- After allowing the port in UFW, the dashboard became accessible.

## Key Takeaway

When adding new features to a hardened server, it is important to remember the security rules that were already put in place earlier.

Otherwise, troubleshooting becomes a conversation between your current self and your past self, and your past self is annoyingly consistent.
