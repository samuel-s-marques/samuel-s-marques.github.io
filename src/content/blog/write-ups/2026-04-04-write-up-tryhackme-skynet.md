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


