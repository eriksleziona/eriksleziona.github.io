---
layout: post
title: "Troubleshooting: Ruby, MSYS2, and PowerShell Execution Policies"
date: 2026-04-10
categories: [Troubleshooting, Windows]
tags: [Ruby, Jekyll, PowerShell, MSYS2]
summary: "A practical recovery path for fixing missing MSYS2 tooling and temporary PowerShell policy blocks during a Windows Jekyll setup."
---

## The Problem

While setting up a Jekyll environment on Windows 11, the installation of gems failed due to missing native compilation tools. The error message was clear:
`MSYS2 could not be found. Please run 'ridk install'.`

However, attempting to run `ridk install` triggered a secondary security roadblock:
`File [...] cannot be loaded because running scripts is disabled on this system.`

## Root Cause Analysis

1. **Toolchain Absence:** Jekyll gems often require C extensions. Without MSYS2 and the MinGW toolchain, Windows cannot compile these gems.
2. **PowerShell Security:** Windows restricts the execution of `.ps1` scripts by default under the `Restricted` execution policy to prevent unauthorized script execution.

## The Solution: A Step-by-Step Recovery

### 1. Bypassing Execution Policy

Instead of permanently lowering the system's security, I used a scoped command to allow the installation script to run only for the current session:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
```

### 2. Installing the Build Tools

1. With the policy adjusted, I executed the Ruby installer helper:

```powershell
ridk install
```

I selected **Option 3 (MSYS2 and MINGW development toolchain)**. This installed gcc, make, and other essential headers.

### 3. Refreshing the Environment

After the toolchain install completed, I reopened PowerShell so the refreshed environment variables and build tools were available before retrying the gem installation.

## Outcome

The environment was successfully repaired, allowing for a clean installation of Jekyll and Bundler.

**Key Takeaway**: As an admin, understanding the "scope" of your changes (process vs. system-wide) is vital for maintaining a secure yet functional workstation.
