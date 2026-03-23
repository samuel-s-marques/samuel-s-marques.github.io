---
author: Samuel Marques
pubDatetime: 2026-03-22
modDatetime: 2026-03-22
title: "Write-up: TryHackMe - Missing Person"
ogImage: "Write-up: TryHackMe - Missing Person"
slug: missing-person
featured: false
draft: false
tags:
  - osint
  - tryhackme
  - easy
description: The TryHackMe "Missing Person" challenge is an OSINT (Open Source
  Intelligence) investigation exercise where participants must track down a
  fictional missing individual using publicly available information.
---
This [OSINT (Open Source Intelligence) challenge](https://tryhackme.com/room/missingperson) simulates an easy missing person investigation. The goal is to analyze publicly available data (photos, social media posts, and online accounts) to track the individual's last known activities and connections.


| Tool | Purpose |
| --------------------------------- | ----------------------------------- |
| Google Image Search | identifying the restaurant |
| [EXIF.tools](https://exif.tools/) | Extracting timestamps from an image |


### The Journey Begins

The investigation starts with holiday photos. At first glance, they look ordinary, just two pics. One from a MotoGP event. The other, from a restaurant. The restaurant photo is our first breadcrumb.

If we do a reverse search on the image, we find out it was taken in **Cantina Mexicana Kuta Lombok**, in Indonesia. Now, if we Google "MotoGP 2025 Indonesia", we can also find which circuit held the race and when. The [Mandalika International Street Circuit](https://en.wikipedia.org/wiki/Mandalika_International_Street_Circuit). By knowing the missing person went to a MotoGP event, and the restaurant he went, we got the answer to the first three questions: "What is the commercial name of this circuit?" (**Pertamina Mandalika International Street Circuit**), "When did the event take place?" (03-05/10/2025), and "What is the name of the restaurant?" (**Cantina Mexicana**).

![image.png](/image.png)

### Following the Clues

We already know the restaurant, the event, and the day, but we need to know when the missing person went to the restaurant. We can obtain this information from its metadata. Websites like [EXIF.tools](https://exif.tools/) can reveal information hidden in the image, such as when it was taken and, in some cases, where it was taken. And the metadata hidden in the photo reveals the exact timestamp: 19:55:30. Suddenly, what looked like a casual dinner becomes a precise moment in our timeline.

![image.png](/image-1.png)

### The After Party

The missing person mentions attending a MotoGP after-party. By googling for "motogp 2025 indonesia after party", we can find on social media the *biggest party* after the Moto GP race, at Surfer's Bar Kuta Lombok. Digging deeper, we uncover the bar's full address in Lombok Tengah (**Jl. Raya Kuta, Kuta, Kec. Pujut, Kabupaten Lombok Tengah, Nusa Tenggara Bar**). And after watching the video, we discover a new character in the story, a DJ known as **Bong Leleh**. A potential lead.

![image.png](/image-2.png)

![A screenshot of surfersbar.lombok's post, showing current DJs.](/screenshot-surfersbar.png)

![image.png](/image-5.png)

### Into the Cave

Tracing Bong Leleh's online presence, through Facebook, reveals his side hustle: guiding tourists to **Gua Sumur**, a cave tucked away in the region. His business contact number, **85333137345**, becomes the final piece of the puzzle, answering the questions for "What caves does he take tourists to?" and "What number did the DJ list for his tour business?" With this, we've reconstructed the missing person's movements, connections, and likely whereabouts.

![image.png](/image-4.png)

### The Final Thoughts

The investigation demonstrates how OSINT techniques, such as analyzing metadata, geolocation, and social media, can piece together a person's trail. Each clue builds upon the last, ultimately reconstructing the person’s movements and contacts. It's a reminder of how much we can leave behind in the digital world.