---
author: Samuel Marques
pubDatetime: 2026-04-05
modDatetime: 2026-04-05
title: "Write-up: TryHackMe - Skynet"
ogImage: "Write-up: TryHackMe - Skynet"
slug: skynet
featured: false
draft: true
tags:
  - exploit
  - tryhackme
  - easy
  - vulnerable vm
  - smb
description: "Skynet: A Terminator-themed Linux machine."
---

| Tool | Purpose |
| ------ | ---------------------------------------------- |
| Nmap | Scanning open ports and services |
| hydra | Bruteforcing `/squirrelmail` auth panel |
| ffuf | Fuzzing directories |
| Netcat | Catching the reverse shell from the web server |


## Reconnaisance

```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Skynet
|_http-server-header: Apache/2.4.18 (Ubuntu)
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: TOP CAPA RESP-CODES PIPELINING UIDL AUTH-RESP-CODE SASL
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: IDLE Pre-login more have LOGIN-REFERRALS LOGINDISABLEDA0001 post-login ENABLE listed capabilities LITERAL+ OK SASL-IR ID IMAP4rev1
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Directory Fuzzing

```
admin           [Status: 301]
config          [Status: 301]
css             [Status: 301]
js              [Status: 301]
squirrelmail    [Status: 301]
index.html      [Status: 200]
```

### SMB

```
Disk              Permissions     Comment
----              -----------     -------
print$            NO ACCESS       Printer Drivers
anonymous         READ ONLY       Skynet Anonymous Share
milesdyson        NO ACCESS       Miles Dyson Personal Share
IPC$              NO ACCESS       IPC Service (skynet server (Samba, Ubuntu))
```

![Captura de tela 2026-04-04 234250.png](</Captura de tela 2026-04-04 234250.png>)

![Captura de tela 2026-04-04 234425.png](</Captura de tela 2026-04-04 234425.png>)

## Exploitation

## Post-Exploitation

