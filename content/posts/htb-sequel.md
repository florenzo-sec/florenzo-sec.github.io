---
title: "HTB Sequel: MariaDB Root Login Without a Password"
date: 2026-10-17
draft: false
tags: ["hackthebox", "starting-point", "mariadb", "mysql", "linux", "very-easy", "weak-credentials"]
categories: ["Write-up"]
summary: "Write-up of Sequel (HTB Starting Point): a MariaDB instance reachable over the network with an empty root password, and how to prevent it."
ShowToc: true
---

**Platform:** Hack The Box, Starting Point · **OS:** Linux · **Difficulty:** Very Easy · **Theme:** MariaDB root account with a blank password

## Summary

The machine exposes a MariaDB service that accepts logins as `root` with no password at all. Connecting with the official client and browsing the available databases is enough to find a non-default database holding the flag in one of its tables.

## Reconnaissance

```bash
nmap <IP> -n -p- -oN scan
```

The scan shows port **3306/tcp** open, running MariaDB.

## Exploitation

I connected with the MariaDB client:

```bash
mariadb -u root -h <IP> --skip-ssl
```

By default the client tries to negotiate SSL/TLS first, and this server did not accept it, so `--skip-ssl` is needed to fall back to a plain connection. No password was requested for `root`.

Once inside, I enumerated the databases:

```sql
SHOW DATABASES;
```

Besides the three default ones (`information_schema`, `mysql`, `performance_schema`), one more database was present: `HTB`. Inside it, a table called `config` held the flag needed to complete the machine:

```sql
USE HTB;
SHOW TABLES;
SELECT * FROM config;
```

## Defense

**Root causes**

- The `root` account has an empty password (`authentication_string` not set / blank).
- The database service is reachable from the network instead of being bound to localhost only.

**Prevention**

- Never leave the `root` password blank; set a strong password immediately after installation (`mysql_secure_installation` or `ALTER USER`).
- Avoid exposing privileged accounts like `root` over the network at all: restrict them to local access (`root@localhost`) and create dedicated, least-privilege users for each application, each with their own credentials and grants scoped to the databases they actually need.
- Bind the service to `127.0.0.1` (or an internal interface) with `bind-address`, and filter port 3306 with a firewall so only authorized application hosts can reach it.
- Enforce TLS for connections that cross untrusted networks, and remove anonymous/test accounts and default databases that aren't needed in production.

**Detection**

- Monitor inbound connections to port 3306 from unexpected sources.
- Log and alert on successful authentications for `root` or other administrative accounts coming from outside the application's own hosts.
- Alert on logins that negotiate without TLS when TLS is expected to be enforced.

**Real-world impact**

A `root` account reachable over the network with no password gives an attacker full control of the database: reading, modifying or deleting any data, and potentially further access to the host depending on the DBMS privileges available (e.g. reading/writing files through `LOAD_FILE`/`INTO OUTFILE` style primitives, where enabled).

## What I learned

Sequel reinforced how often "it's just the default install" turns into a critical exposure: a blank root password plus network exposure is enough to lose the whole database. I also learned that the MariaDB client tries SSL by default and that `--skip-ssl` is sometimes necessary when the server itself doesn't support or offer it.
