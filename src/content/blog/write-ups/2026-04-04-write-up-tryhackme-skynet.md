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
[Skynet](https://tryhackme.com/room/skynet) is a beginner-to-intermediate CTF that blends enumeration, web exploitation, brute force, and privilege escalation. The goal is to recover `user.txt` and `root.txt` flags.

> **Note:** You may notice IP addresses shifting throughout this walkthrough (e.g., `10.66.145.158` to `10.66.174.94`). This is due to a machine restart during the operation.


| Tool | Purpose |
| ------ | ---------------------------------------------- |
| Nmap | Scanning open ports and services |
| hydra | Bruteforcing `/squirrelmail` auth panel |
| ffuf | Fuzzing directories |
| Netcat | Catching the reverse shell from the web server |
| SMBMap | Mapping SMB share disks |


## Environment and Goal

We are operating within a controlled lab environment provided by **TryHackMe**. To access the target, we establish a **Private VPN Tunnel** using OpenVPN, which places our attacking machine (Kali Linux) on the same virtual network as the target.

- Attacker Machine:
  - Distribution: Kali Linux 2026.1
  - Purpose: Used as the penetration testing platform, equipped with tools such as Nmap, Metasploit Framework, and other utilities for reconnaissance and exploitation.
- Target Machine:
  - Distribution: Skynet (Ubuntu-based vulnerable VM)
  - Purpose: Capture the flags (keys) by exploiting weaknesses in the system

## Reconnaisance

Every successful infiltration begins with seeing what's behind the curtain. Our first move is a comprehensive **Nmap** scan to identify the attack surface.

```
nmap -sC -sV -T4 10.66.145.158
```

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

The scan reveals 6 different ports:

- **22 (SSH)**: Remote administration
- **80 (HTTP)**: A web server
- **110/143 (POP3/IMAP)**: Email services
- **139/445 (SMB)**: Samba file-sharing (important)

### Directory Fuzzing

![image.png](/image-35.png)

Navigating to the IP in a browser shows a Skynet search engine. It looks innocent, but a directory brute-force attack using `ffuf` reveals a hidden directory: `/squirrelmail`.

```
admin           [Status: 301]
config          [Status: 301]
css             [Status: 301]
js              [Status: 301]
squirrelmail    [Status: 301]
index.html      [Status: 200]
```

![Captura de tela 2026-04-04 233855.png](</Captura de tela 2026-04-04 233855.png>)

### SMB

While SquirrelMail is tempting, we lack credentials. Let's pivot to the Samba shares to see if Skynet left any digital breadcrumbs.

Using `smbmap`, we look for accessible shares:

```
smbmap -H 10.66.145.158
```

```
Disk              Permissions     Comment
----              -----------     -------
print$            NO ACCESS       Printer Drivers
anonymous         READ ONLY       Skynet Anonymous Share
milesdyson        NO ACCESS       Miles Dyson Personal Share
IPC$              NO ACCESS       IPC Service (skynet server (Samba, Ubuntu))
```

We hit paydirt: an **anonymous** share with read-only permissions. We connect without a password.

![Captura de tela 2026-04-04 234250.png](</Captura de tela 2026-04-04 234250.png>)

Inside, we find a message from **Miles Dyson** in `attention.txt` mentioning a system-wide password reset, along with a directory of log files. After downloading and inspecting `log1.txt`, we find what looks like a wordlist of potential passwords.

![Captura de tela 2026-04-04 234425.png](</Captura de tela 2026-04-04 234425.png>)

## Exploitation

## Post-Exploitation

