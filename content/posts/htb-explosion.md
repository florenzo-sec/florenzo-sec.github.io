---
title: "HTB Explosion: RDP With a Blank Administrator Password"
date: 2026-10-16
draft: false
tags: ["hackthebox", "starting-point", "rdp", "windows", "very-easy", "weak-credentials", "misconfiguration"]
categories: ["Write-up"]
summary: "Write-up of Explosion (HTB Starting Point): RDP exposed with an Administrator account that has no password, and how to prevent and detect it."
ShowToc: true
---

**Platform:** Hack The Box, Starting Point · **OS:** Windows · **Difficulty:** Very Easy · **Theme:** RDP exposed with weak credentials

## Summary

The machine exposes Remote Desktop (RDP) on its default port, and the built-in `Administrator` account has a blank password. Connecting with an RDP client and no password is enough to get a full graphical session as administrator.

## What is RDP?

Remote Desktop Protocol (RDP) is Microsoft's protocol for controlling a Windows machine remotely through its graphical interface. It listens on TCP port 3389 by default and shows up in scans as `ms-wbt-server`. It is convenient for administrators and, for the same reason, a very common target: a reachable RDP service with weak credentials gives an attacker the same access as sitting at the keyboard.

## Reconnaissance

```bash
nmap -n <IP> -oN first
```

- `-n` disables DNS resolution, which makes the scan faster.
- `-oN first` saves the output in normal format to a file called `first`.

The scan shows port **3389/tcp** open, with the service `ms-wbt-server` (RDP).

This is the default scan, so it only covers the 1000 most common ports. That was enough here, since RDP runs on its standard port, but on a real target a full scan (`-p-`) and version detection (`-sV`) are worth adding.

## Exploitation

I tested the login with `xfreerdp`, using the `Administrator` account and no password:

```bash
xfreerdp /v:<IP> /u:administrator
```

When the client asked for a password, the blank one was accepted and I landed in the Windows desktop (GUI) as administrator. No exploit was needed: the weakness is the configuration.

## Defense

**Root causes**

- The `Administrator` account has a blank password.
- RDP is reachable from the network where the attacker is.

A note worth remembering: on a default Windows installation, the policy *"Accounts: Limit local account use of blank passwords to console logon only"* is enabled, which blocks network logons (including RDP) for accounts with a blank password. For this attack to work, that protection must have been turned off. Checking this setting is a good hardening step.

**Prevention**

- Never leave any account, least of all `Administrator`, with a blank or weak password. Enforce a password policy with sufficient length and complexity.
- Do not expose port 3389 to untrusted networks. Put RDP behind a VPN or a Remote Desktop Gateway, and restrict it with firewall rules to known source addresses.
- Keep Network Level Authentication (NLA) enabled, so authentication happens before a session is created.
- Use multi-factor authentication for remote access where possible.
- Rename or disable the built-in `Administrator` account and use named accounts with least privilege.
- Configure account lockout to slow down password guessing.

**Detection**

- Monitor Windows Security events for successful logons (event ID 4624; RDP appears as logon type 10 or, with NLA, a network logon first) and failed ones (4625).
- Alert on logons to the `Administrator` account from remote addresses, on bursts of failed logons, and on connections to port 3389 from unexpected sources.

**MITRE ATT&CK mapping**

- T1021.001: Remote Services, Remote Desktop Protocol
- T1078.001: Valid Accounts, Default Accounts

## What I learned

The main takeaway from Explosion is simple: never leave the administrator password blank.
