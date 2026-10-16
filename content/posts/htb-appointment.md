---
title: "HTB Appointment: Bypassing a Login With SQL Injection"
date: 2026-10-16
draft: false
tags: ["hackthebox", "starting-point", "sql-injection", "web", "very-easy", "owasp-top-10"]
categories: ["Write-up"]
summary: "Write-up of Appointment (HTB Starting Point): a login form vulnerable to SQL injection, bypassed with a comment character, and how to prevent it with prepared statements."
ShowToc: true
---

**Platform:** Hack The Box, Starting Point · **OS:** Linux · **Difficulty:** Very Easy · **Theme:** SQL injection (authentication bypass)

## Summary

The machine hosts a web application with a login form. The form builds a SQL query directly from user input, with no sanitization, which makes it vulnerable to SQL injection (OWASP Top 10 2021, A03:2021-Injection). By injecting a comment character in the username field, it's possible to neutralize the password check in the underlying query and log in as `admin` without knowing the password.

## Reconnaissance

```bash
nmap -n -oN first <IP>
```

The default scan shows port **80/tcp** open, service `http`. A follow-up scan adds version detection:

```bash
nmap -n -p 80 -sV -oN http <IP>
```

```text
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 (Debian)
```

The web server is Apache 2.4.38 on Debian, hosting a login page.

## Exploitation

The login form sends a standard POST request with two parameters, `username` and `password`. An application that builds its SQL query by directly concatenating these values (instead of using parameterized queries) is vulnerable to SQL injection: user input can break out of the intended string and alter the query's logic.

In MySQL, a line can be commented out not only with `--` (followed by a space) but also with a single `#` character — a detail that isn't true for every database engine, and worth remembering when moving between them.

By submitting a username ending in a quote followed by `#`, the rest of the query — including the password check — gets commented out, and the query only verifies that a user named `admin` exists:

```
username = admin'#
password = (anything)
```

This is enough to authenticate as `admin` without knowing the real password, landing on the page with the flag.

## Defense

**Root cause**

User input is concatenated directly into a SQL query instead of being treated strictly as data.

**Prevention**

- Use **prepared statements / parameterized queries** everywhere user input reaches a query. The query structure is compiled first, and the input is then bound as data — it can never change the query's logic, no matter what characters it contains.
- Use an ORM with parameterized queries by default, rather than hand-built SQL strings.
- Apply input validation as a second layer (expected format, length, character set), never as the only defense.
- Run the database account used by the web app with least privilege: no access to databases, tables, or operations it doesn't need.
- Add a Web Application Firewall (WAF) as an extra layer, not a replacement for fixing the code.

**Detection**

- Log failed and successful authentication attempts, and alert on logins containing SQL metacharacters (`'`, `"`, `#`, `--`, `;`) in username or password fields.
- Monitor the database's own query logs for anomalous or malformed queries coming from the application.
- A WAF or IDS with SQL injection signatures can catch many common patterns, though it shouldn't be relied on as the sole control.

**MITRE ATT&CK mapping**

- T1190: Exploit Public-Facing Application

## What I learned

Not every database comments out the rest of a line the same way: MySQL accepts both `--` (with a trailing space) and `#`. And the real fix for this class of bug is architectural, not a filter: prepared statements separate the query structure from the data, so user input is never interpreted as part of the query logic in the first place.
