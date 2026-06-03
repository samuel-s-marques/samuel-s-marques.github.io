---
author: Samuel Marques
pubDatetime: 2026-06-03
modDatetime: 2026-06-03
title: "Writeup: TwoMillion - HackTheBox"
ogImage: "Writeup: TwoMillion - HackTheBox"
slug: twomillion
featured: true
draft: true
tags:
  - exploit
  - ctf
  - easy
  - hackthebox
description: Our company may have been compromised, we need your help ASAP.
---
[HackTheBox's TwoMillion](https://app.hackthebox.com/machines/TwoMillion) is an Easy difficulty Linux box that was released to celebrate reaching 2 million users on HackTheBox.

# Environment and Goal

Hey there! Before we dive into the nitty-gritty details of how we broke into this system, let's set the stage. Every penetration test starts with a good understanding of what we’re dealing with — the environment — and knowing exactly what we are trying to achieve — the goal.

Our target for this exercise was **HackTheBox's TwoMillion**. We were testing an older iteration of their platform, one built to celebrate a major community milestone. This immediately signaled that we might be dealing with some legacy code or architecture, which is often where security flaws hide.

# Reconnaissance

Our first step was always reconnaissance — seeing what doors were open and what technologies were running behind them. By performing initial scans (like a Rustscan scan), we quickly identified two primary attack vectors visible to the public:

1.  **HTTP Web Server (Port 80):** Running a standard web service, likely powered by Nginx. This is our main point of entry for any web-related flaws.
2.  **SSH Service (Port 22):** Providing secure shell access, which could indicate direct system administration interfaces were available.

*(See this screenshot for a visual overview of the open ports.)*
![Screenshot of Rustscan showing two open ports: 22 (SSH) and 80 (HTTP).](Image Evidence - Captura de tela 2026-06-03 091808.png)

Beyond these initial findings, we spent time exploring the web application itself. The platform looked like a classic educational hacking environment—the older the system, the more potential weak spots there are! We used directory brute-forcing tools to map out all available API endpoints and user-facing sections, uncovering directories like `/invite`, `/login`, and `/register`.

## JavaScript Code Analysis

Once we had mapped out key directories like `/api`, `/login`, and `/invite`, our attention shifted from simply looking at page structures to digging into the source code itself. Modern web applications rarely rely on simple static HTML; they do the heavy lifting using JavaScript.

We focused specifically on a minified JavaScript file: **`inviteapi.min.js`**. This file is what powers the functionality of the invitation system.

**The Obfuscation Challenge:**
When we opened these files, they were heavily obfuscated—a common practice to make reverse engineering difficult. We identified a pattern known as a "Dean Edwards Packer", which is essentially an old-school way of compressing and hiding executable code.

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

To proceed, we had to 'de-minify' the JavaScript. This process involved reversing the packer's encryption/compression layer. A critical step here was identifying where sensitive commands were being executed (like functions that used `eval()`) and replacing them with visible logging statements (`console.log`). This simple act of cleaning up the code allowed us to finally read the underlying logic.



**Unmasking the API Endpoints:**
By reading the clean JS, we didn't find a vulnerability—we found a *blueprint*. The JavaScript contained explicit function calls for managing invite codes: `verifyInviteCode` and `makeInviteCode`. These functions weren’t just placeholders; they were making POST requests to specific API endpoints that were completely hidden from a standard user view.

Specifically, the code revealed how to generate an invite code by hitting `/api/v1/invite/generate`, confirming a crucial piece of missing information about the system's underlying architecture.

### Deep Dive into the Invite Page (`/invite`)

The `/invite` directory proved to be our primary initial vector. While it looked like just another user-facing page, its functionality was tied directly to the hidden API endpoints we discovered in the JavaScript code.

**Initial Flaw Discovery:**
When we analyzed how the platform handled invite codes, we realized that the system had a design flaw: the mechanisms for *generating* and *verifying* these codes were separate from the standard user login process. The entire workflow was dictated by those API calls.

The fact that an older, more accessible version of this invitation mechanism existed meant that it likely contained deprecated security practices. Instead of relying on users to navigate a complex flow, we could interact directly with the exposed APIs.

This successful reconnaissance phase allowed us to move from being external observers (who simply *view*ed the website) to active participants who knew exactly which hidden system buttons needed to be pressed in sequence: **First**, exploit the invite code weakness for initial entry; and **second**, leverage that newly created user role to access higher-privilege API endpoints.

---
*(The flow naturally transitions from understanding the weaknesses of application logic (JS/API) into exploiting them, which is what was covered in "Exploitation.")*

# Exploitation

Our objective wasn't just to get a simple shell. Given the nature of the platform and its history, we aimed for a comprehensive system compromise by exploiting multiple layers of weakness:

**Phase 1: Initial Beachhead (Low-Privilege Access)**
The core goal here was to gain initial access using application logic flaws. We noticed an old feature related to **invite codes**. By analyzing JavaScript files within the platform's source code, we determined that there was a vulnerability in how invite codes were handled. This allowed us to bypass standard login procedures and create a low-privilege account on the system.

**Phase 2: Privilege Escalation (Admin Access)**
Once inside as a regular user, our goal shifted from simple access to **escalating privileges**. We successfully manipulated the application's API endpoints—specifically related to VPN generation—to elevate our limited user role all the way up to **Administrator**. This was a crucial step because Admin roles typically have access to highly sensitive system functions.

**Phase 3: Total System Control (Root Shell)**
With administrative power, we were in a prime position to execute devastating attacks. Our final goals included:

*   **Command Injection:** Exploiting the admin VPN generation endpoint, which allowed us to inject and run arbitrary commands directly on the underlying server.
*   **Kernel Exploitation:** Finally, by identifying an outdated system kernel version, we utilized a known vulnerability (the CVE-2023-0386) to bypass all software controls and gain complete **root access**—total control over the entire operating system.

In short? We moved from simply viewing an old website to fully compromising its underlying Linux kernel by chaining together a series of application, privilege, and OS vulnerabilities.

# Post-Exploitation