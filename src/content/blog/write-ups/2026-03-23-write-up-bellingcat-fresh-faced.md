---
author: Samuel Marques
pubDatetime: 2026-03-23
modDatetime: 2026-03-23
title: "Write-up: Bellingcat - Fresh Faced"
ogImage: "Write-up: Bellingcat - Fresh Faced"
slug: fresh-faced
featured: false
draft: false
tags:
  - osint
  - bellingcat
  - easy
description: The Bellingcat "Fresh Faced" challenge was to locate Eliot Higgins
  in a YouTube video from his early days, before Bellingcat became widely known.
---
The [challenge](https://challenge.bellingcat.com/) asked us to track down **Eliot Higgins**, the founder of Bellingcat, in one of his earliest public appearances, a YouTube documentary. The hint was clear: his early footprint was out there, but buried beneath years of uploads and algorithm noise.

![image.png](/image-22.png)

### Approach

The key to solving this challenge was leveraging **Google Dorks** — advanced search operators that refine queries beyond standard keyword searches. By combining filters for site, date ranges, and keywords, we can uncover content that would otherwise be buried

We knew the subject was **Eliot Higgins**, and the challenge implied looking into his early online footprint from 2013.

We used the following query:

```
eliot higgins site:youtube.com before:2013-12-31 after:2013-01-01
```

- `eliot higgins` -> keyword to locate his name
- `site:youtube.com` -> restricts results to YouTube
- `before: 2013-12-31 after: 2013-01-01` -> narrows results to videos uploaded in 2013

Running the dork revealed a **documentary uploaded in 2013** featuring Higgins, solving the Fresh Faced challenge. Watching the video confirmed his appearance, the exact solution the challenge was pointing us toward.

![image.png](/image-23.png)

&nbsp;