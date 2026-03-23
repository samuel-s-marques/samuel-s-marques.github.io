---
author: Samuel Marques
pubDatetime: 2026-03-23
modDatetime: 2026-03-23
title: "Write-up: TryHackMe - OhSINT"
ogImage: "Write-up: TryHackMe - OhSINT"
slug: ohsint
featured: false
draft: false
tags:
  - osint
  - tryhackme
  - easy
description: "Open-source intelligence (OSINT) challenges are a fantastic way to
  sharpen investigative skills. The OhSINT room on TryHackMe is a classic
  example: starting with a single image, we unravel a trail of digital
  breadcrumbs across platforms. Let’s dive into the process step by step."
---
Open-source intelligence (OSINT) challenges are a fantastic way to sharpen investigative skills. The **[OhSINT room on TryHackMe](https://tryhackme.com/room/ohsint)** is a classic example: starting with a single image, we unravel a trail of digital breadcrumbs across platforms. Let’s dive into the process step by step.


| Tool | Purpose |
| ------------------------------------------------------- | -------------------------------- |
| [EXIF.tools](https://exif.tools/) | Extracting usernames from a file |
| [WiGLE](https://wigle.net/) | Finding the location by a BSSID |
| [Instant Username Search](https://instantusername.com/) | Finding the user's social media |


### The Image

The challenge begins with an image file. At first glance, it looks ordinary, just the Windows XP wallpaper, but metadata often hides secrets.

![](/WindowsXP_1551719014755.jpg)

If we open the image in [EXIF.tools](https://exif.tools/), we can see a username: **OWoodflint**. This is our first lead.

![image.png](/image-29.png)

### Tracing the Username

Usernames are often reused across platforms. To check this, we can use [instantusername.com](https://instantusername.com/?q=OWoodflint). We search for OWoodflint, and we find out there's a [Twitter/X account](https://xcancel.com/OWoodflint) under that name.

We see that the profile picture is a **cat**, and the bio mentions **an interest in open-source projects**.

![image.png](/image-30.png)

### The Wi-Fi Clue

Scrolling through OWoodflint’s posts, one tweet stands out:

> From my house I can get free wifi ;D  
>
> Bssid: B4:5D:50:AA:86:41 - Go nuts!

![image.png](/image-31.png)

If we search the BSSID on [WiGLE](https://wigle.net/), we see that the location resolves to **London**, with SSID **UnileverWiFi**.

![Captura de tela 2026-03-23 115757.png](</Captura de tela 2026-03-23 115757.png>)

### GitHub Connection

The X bio mentions open source, but no GitHub link is provided. But if we Google the username, we find a repository: [OWoodfl1nt/people_finder](https://github.com/OWoodfl1nt/people_finder?tab=readme-ov-file). Inside, we find both an **email address** and a link to [a WordPress blog](https://oliverwoodflint.wordpress.com/).

### The Blog

Following the blog link, we uncover more personal details: that the user is in **New York**, and a password, **"pennYDr0pper.!**".

![image.png](/image-32.png)

&nbsp;