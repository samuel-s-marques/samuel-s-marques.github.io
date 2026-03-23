---
author: Samuel Marques
pubDatetime: 2026-03-22
modDatetime: 2026-03-22
title: "TryHackMe: Dev Diaries"
ogImage: "TryHackMe: Dev Diaries"
slug: dev-diaries
featured: false
draft: false
tags:
  - osint
  - tryhackme
  - easy
description: The TryHackMe "Dev Diaries" challenge is an OSINT and code
  forensics exercise where participants trace a developer’s digital footprint
  across different platforms.
---
## Write-up: TryHackMe - Dev Diaries

One of the most fascinating aspects of OSINT and security research is how much information can be uncovered simply by following digital breadcrumbs.

### Mapping the Domain

The journey began with **subdomain enumeration**. Using [Subdomain Finder](https://subdomainfinder.c99.nl/), I scanned the domain [marvenly.com](http://marvenly.com) and uncovered two subdomains. Among them was a development environment (uat-testing.marvenly.com). Unfortunately, direct access wasn't possible, it was locked down. I don't know if it was on purpose.

![image.png](/image-7.png)

### The Wayback Machine

When doors are closed, archives often hold the key. Turning to the [Wayback Machine](https://web.archive.org/), I found a snapshot of the UAT site: 

![image.png](/image-6.png)

![image.png](/image-8.png)

From this, I extracted a **username** that became the next lead: **notvibecoder23**.

### GitHub Recon

Searching the username on GitHub revealed the developer's profile. A single repository, the website, with commits and activity logs, all waiting to be explored.

I picked a [random commit](https://github.com/notvibecoder23/marvenly_site/commit/7a7090dd0ce6b8932d0c4a44e050e7fa1e0b2edd.patch) and appended `.patch` to the URL. This trick exposed the **developer's email address**, hidden in the commit metadata: **freelancedevbycoder23@gmail.com.**

### Git History

Now with only two questions left, I dug deeper into the **git history**. Commit messages and diffs revealed more personal and technical details about the developer's workflow and oversights. 

The second commit, 88baf1d, reveals to us why the project was abandoned: **[The project was marked as abandoned due to a payment dispute.](https://github.com/notvibecoder23/marvenly_site/commit/88baf1db29d7530a51c7bc13ae9f3c1b9a1eae25 "The project was marked as abandoned due to a payment dispute")** And the [third commit](https://github.com/notvibecoder23/marvenly_site/commit/33c59e5feedcbcbfee7a1f6d3a435225698f616f), about removing the signature, revealed the flag: **THM{g1t_h1st0ry_n3v3r_f0rg3ts}**.

### Lessons Learned

This challenge highlights a crucial reality:

- Subdomains can expose sensitive environments.
- Archived content may reveal information long after it's been removed.
- Git commits are permanent records, emails usernames, and even mistakes live on in history.

Operational security (OPSEC) isn't just about servers. It's about every trace left online.