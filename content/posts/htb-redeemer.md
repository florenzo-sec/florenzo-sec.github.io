---
title: "HTB Redeemer: Redis esposto senza autenticazione"
date: 2026-10-16
draft: false
tags: ["hackthebox", "starting-point", "redis", "linux", "very-easy"]
categories: ["Writeup"]
summary: "Writeup di Redeemer (HTB Starting Point): un'istanza Redis raggiungibile dalla rete e senza password, e come si blinda."
ShowToc: true
---

**Piattaforma:** HackTheBox, Starting Point · **OS:** Linux · **Difficoltà:** Very Easy · **Tema:** Redis esposto senza autenticazione

## Sintesi

La macchina espone un'istanza Redis raggiungibile dalla rete e priva di autenticazione. Basta connettersi con il client ufficiale, enumerare le chiavi e leggere quella che contiene la flag.

## Cos'è Redis

Redis è un database NoSQL *in-memory*: i dati risiedono nella RAM invece che su disco, quindi letture e scritture sono molto veloci. Per questo si usa soprattutto come cache, ma anche per sessioni, code e contatori. Funziona con un modello chiave-valore e ascolta di default sulla porta TCP 6379.

## Ricognizione

```bash
nmap <IP> -n -p- -oN scan
```

- `-n` disabilita la risoluzione DNS, rendendo la scansione più rapida.
- `-p-` scansiona tutte le 65535 porte TCP, invece delle sole 1000 più comuni di default.
- `-oN scan` salva l'output in formato normale nel file `scan`.

Il risultato mostra la porta **6379/tcp** aperta con servizio Redis.

Scansionare tutte le porte è una buona pratica: la 6379 è quella standard e sarebbe stata trovata anche con lo scan di default, ma `-p-` permette di scoprire servizi su porte non convenzionali, che in un contesto reale non sono rari. Per identificare con precisione servizio e versione si può aggiungere `-sV`; qui la versione l'ho ricavata direttamente da Redis con `INFO`.

## Exploitation

Connessione con il client ufficiale:

```bash
redis-cli -h <IP>
```

Una volta dentro, senza che venga richiesta alcuna password:

```text
INFO            # versione del server e configurazione
SELECT 0        # seleziona il database 0
KEYS *          # elenca tutte le chiavi
GET flag        # legge il valore della chiave "flag"
```

`INFO` restituisce, tra le altre cose, la versione di Redis, il sistema operativo e il numero di chiavi per database. `KEYS *` elenca ogni chiave del database selezionato e `GET` ne recupera il valore.

## Difesa

**Cause della vulnerabilità**

- Nessuna autenticazione configurata (`requirepass` assente).
- Servizio esposto su tutte le interfacce invece che solo in locale.

**Prevenzione**

- Impostare una password robusta (`requirepass`) o, meglio, usare le ACL (Redis 6+) con utenti a privilegi minimi.
- Limitare l'ascolto con `bind 127.0.0.1` e lasciare `protected-mode yes`.
- Filtrare la porta 6379 con firewall o segmentazione di rete: solo gli host applicativi autorizzati devono raggiungerla.
- Usare TLS se il traffico attraversa reti non fidate.
- Disabilitare o rinominare i comandi pericolosi (`KEYS`, `CONFIG`, `FLUSHALL`).

**Rilevazione**

- Monitorare connessioni in ingresso sulla 6379 da sorgenti inattese.
- Loggare i comandi e allertare su `KEYS *`, `CONFIG SET` e `INFO` provenienti da host non previsti.

**Rischio reale**

Oltre alla lettura dei dati, un Redis aperto e scrivibile può portare a esecuzione di codice (ad esempio con `CONFIG SET dir` e `dbfilename` per scrivere file arbitrari). Qui non è stato necessario, ma è l'impatto potenziale da tenere presente.

## Cosa ho imparato

Redeemer è stato il mio primo contatto con Redis e con `redis-cli`. Prima di questa macchina non sapevo cosa fosse Redis né a cosa servisse: ho scoperto che è un database in-memory, usato soprattutto per il caching, e ho imparato a interrogarlo con i comandi base (`INFO`, `SELECT`, `KEYS`, `GET`).
