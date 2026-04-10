---
layout: page
title: About
permalink: /about/
intro: A living portfolio for the systems, notes, and operating habits I am building through hands-on lab work.
---

## About me

{{ site.profile_headline }}

{{ site.profile_short }}

{{ site.profile_story }}

## What I am focused on right now

{% for item in site.profile_focus %}
- {{ item }}
{% endfor %}

## What I use this lab for

- Turn theory into repeatable hands-on practice.
- Create troubleshooting notes that explain root cause, recovery, and lessons learned.
- Build a portfolio that shows process, not just final screenshots.

## Current stack

- Hypervisors and platforms: `{{ site.main_hypervisor }}`
- Current status: `{{ site.lab_status }}`
- Main focus: Linux, Python, and practical systems administration in hands-on lab environments.

## What you will find here

- Build logs and architecture decisions from the lab.
- Troubleshooting write-ups with practical recovery steps.
- Notes on security, scripting, and infrastructure experiments that I can reuse later.

## What I want this portfolio to show

{% for item in site.profile_goals %}
- {{ item }}
{% endfor %}

## Outside the lab

I care a lot about making technical work understandable. The goal is not just to fix systems, but to explain them clearly enough that the next fix is faster and safer.

You can also find me on [GitHub](https://github.com/{{ site.github_username }}) and [LinkedIn](https://www.linkedin.com/in/{{ site.linkedin_username }}).
