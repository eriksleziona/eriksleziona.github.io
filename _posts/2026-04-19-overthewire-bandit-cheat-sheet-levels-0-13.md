---
layout: post
title: "OverTheWire Bandit: My Technical Cheat Sheet (Levels 0-13)"
date: 2026-04-19 09:00:00 +0200
categories: [Linux, Wargames]
tags: [Bandit, CLI, Security, Documentation]
summary: "A practical cheat sheet for OverTheWire Bandit levels 0 through 13, covering the commands, logic, and command-line habits that every future sysadmin should build early."
---

## OverTheWire Bandit: The SysAdmin's Rite of Passage

Getting comfortable with the Linux Command Line Interface is one of the first real steps for anyone aiming at system administration.

And at some point, somebody always says:

> "You should try OverTheWire Bandit."

Usually they say it with the same expression people use when recommending a workout that will "build character."

They are not wrong.

The **Bandit wargame** is a classic way to build CLI muscle memory, learn how Linux tools behave, and discover that a file named `-` can ruin your mood far more effectively than it has any right to.

Below is my technical cheat sheet for the first 13 levels.

---

## Quick Command Cheat Sheet

| Level | Focus Skill | Key Command |
| :--- | :--- | :--- |
| **0 -> 1** | Basic navigation | `cat readme` |
| **1 -> 2** | Special filenames | `cat ./-` |
| **2 -> 3** | Spaces in names | `cat "file name"` |
| **3 -> 4** | Hidden files | `ls -a` |
| **4 -> 5** | File inspection | `file ./*` |
| **5 -> 6** | Advanced search | `find -size 1033c` |
| **6 -> 7** | System-wide search | `find / -user bandit7` |
| **7 -> 8** | Pattern matching | `grep "millionth"` |
| **8 -> 9** | Unique lines | `sort | uniq -u` |
| **9 -> 10** | Human-readable strings | `strings data.txt` |
| **10 -> 11** | Base64 decoding | `base64 -d` |
| **11 -> 12** | Rot13 cipher | `tr 'A-Za-z' 'N-ZA-Mn-za-m'` |
| **12 -> 13** | Compression and decompression | `tar`, `gzip`, `bzip2` |

---

## Notable Level Breakdowns

### Level 1: Dealing with the Dash `-`

Linux treats a file named `-` as standard input, which is one of those details that makes perfect sense only after it has already wasted your time.

To read it correctly, the path has to be explicit:

```bash
cat ./-
```

That tiny `./` is doing a lot of emotional labor.

### Level 6: Finding Files by Attributes

This level was a great reminder that `find` is not just useful, it is practically a superpower once you know how to combine filters properly.

I had to search the filesystem by:

- file type
- user
- group
- file size

And because Linux enjoys showing permission errors while you are trying to think, redirecting standard error was also part of the solution:

```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

This was one of those moments where the command line stops feeling like a wall of text and starts feeling like a proper tool.

### Level 12: The Decompression Rabbit Hole

This one felt less like a level and more like a small revenge plot written by archive formats.

The biggest lesson here was **file signatures**.
Extensions can lie.
The `file` command usually does not.

So the workflow became:

1. Inspect the file type.
2. Rename it if needed.
3. Use the correct tool.
4. Repeat.
5. Question your life choices just a little.

It was a very practical reminder that troubleshooting often means trusting what the system tells you, not what the filename claims.

---

## Why Am I Documenting This?

Because a sysadmin is only as good as their ability to find, filter, and interpret information quickly.

These levels are not really about passwords.
They are about building habits with tools like:

- `grep`
- `find`
- `sort`
- `uniq`
- `strings`
- `tr`

That is exactly the kind of toolkit that becomes useful later in real Linux administration, troubleshooting, and security work.

So yes, this is a cheat sheet.
But it is also a reminder that every weird little command you learn now saves you time later when a real server is being far less friendly than Bandit.

## Next Step

Onward to **Level 14**, where the game starts getting friendlier with **SSH keys** and less friendly with anyone who hoped the early levels were the whole challenge.
