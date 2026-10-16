---
title: "HTB Redeemer: Redis Exposed Without Authentication"
date: 2026-10-16
draft: false
tags: ["hackthebox", "starting-point", "redis", "linux", "very-easy"]
categories: ["Write-up"]
summary: "Write-up of Redeemer (HTB Starting Point): a Redis instance reachable over the network with no password, and how to lock it down."
ShowToc: true
---

**Platform:** Hack The Box, Starting Point · **OS:** Linux · **Difficulty:** Very Easy · **Theme:** Redis exposed without authentication

## Summary

The machine exposes a Redis instance that is reachable over the network and has no authentication. All it takes is connecting with the official client, listing the keys and reading the one that holds the flag.

## What is Redis?

Redis is an *in-memory* NoSQL database: data lives in RAM instead of on disk, so reads and writes are very fast. That is why it is mostly used as a cache, but it also serves for sessions, queues and counters. It uses a key-value model and listens on TCP port 6379 by default.

## Reconnaissance

```bash
nmap <IP> -n -p- -oN scan
```

- `-n` disables DNS resolution, which makes the scan faster.
- `-p-` scans all 65535 TCP ports instead of only the 1000 most common ones.
- `-oN scan` saves the output in normal format to a file called `scan`.

The result shows port **6379/tcp** open, running Redis.

Scanning every port is good practice: 6379 is the standard Redis port and the default scan would have found it too, but `-p-` also reveals services on non-standard ports, which are far from rare in real environments. To identify the service and its version precisely you can add `-sV`; here I got the version straight from Redis with `INFO`.

## Exploitation

Connect with the official client:

```bash
redis-cli -h <IP>
```

Once connected, no password is requested:

```text
INFO            # server version and configuration
SELECT 0        # select database 0
KEYS *          # list all keys
GET flag        # read the value of the "flag" key
```

`INFO` returns, among other things, the Redis version, the operating system and the number of keys per database. `KEYS *` lists every key in the selected database and `GET` retrieves its value.

## Defense

**Root causes**

- No authentication configured (`requirepass` not set).
- Service exposed on all interfaces instead of localhost only.

**Prevention**

- Set a strong password (`requirepass`) or, better, use ACLs (Redis 6+) with least-privilege users.
- Restrict the listening address with `bind 127.0.0.1` and keep `protected-mode yes`.
- Filter port 6379 with a firewall or network segmentation, so only authorized application hosts can reach it.
- Use TLS when traffic crosses untrusted networks.
- Disable or rename dangerous commands (`KEYS`, `CONFIG`, `FLUSHALL`).

**Detection**

- Monitor inbound connections to port 6379 from unexpected sources.
- Log commands and alert on `KEYS *`, `CONFIG SET` and `INFO` coming from unexpected hosts.

**Real-world impact**

Beyond reading data, an open and writable Redis can lead to code execution (for example, using `CONFIG SET dir` and `dbfilename` to write arbitrary files). That wasn't needed here, but it is the potential impact to keep in mind.

## What I learned

Redeemer was my first hands-on contact with Redis and `redis-cli`. Before this machine I didn't know what Redis was or what it was used for: I found out it is an in-memory database used mainly for caching, and I learned to query it with the basic commands (`INFO`, `SELECT`, `KEYS`, `GET`).
