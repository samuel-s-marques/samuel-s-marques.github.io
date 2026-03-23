---
author: Samuel Marques
pubDatetime: 2026-03-23
modDatetime: 2026-03-23
title: "TryHackMe: Operation Slither"
ogImage: "TryHackMe: Operation Slither"
slug: operation-slither
featured: false
draft: false
tags:
  - osint
  - tryhackme
  - easy
description: The TryHackMe "Operation Slither" challenge is an OSINT challenge,
  where we need to find the people behind an attack.
---
[Operation Slither](https://tryhackme.com/room/operationslitherIU) is a challenge that tests your ability to pivot across platforms, analyze social media clues, and decode hidden messages. The mission: track down the operators of the **Sneaky Viper** group and recover their leaked flags.

## The Leader

### The Forum Post

The investigation began on a hacker forum, where a post advertised stolen data:

```
Full user database TryTelecomMe on sale!!!

As part of Operation Slither, we've been hiding for weeks in their network and have now started to exfiltrate information. 
This is just the beginning. We'll be releasing more data soon. Stay tuned!

@v3n0mbyt3_
```

The handle **@v3n0mbyt3_** was our first lead, potentially the leader. 

### Following the Trail

We tracked down [v3n0mbyt3_ on Twitter](https://xcancel.com/v3n0mbyt3_). Interestingly, he posted "Threads is more fun! Twitter out." That's a clear hint to check his Threads account. So off we go to [https://www.threads.com/@v3n0mbyt3_](https://www.threads.com/@v3n0mbyt3_).

![image.png](/image-17.png)

Scrolling through his [Threads activity](https://www.threads.com/@v3n0mbyt3_/post/C6G35gPPTPU?xmt=AQF0xH9HkpFq4FAlDOUb9jK7gWrDLUDvS6TUWn_rCpsdxw), we stumble upon a curious response from another user. It contains a strange string:

> I really can't get over with this one 🤪
>
> VEhNe3NsMXRoM3J5X3R3MzN0el80bmRfbDM0a3lfcjNwbDEzcyF9

Looks suspiciously like an encoded message. Time to investigate.

![image.png](/image-18.png)

### Identifying and Decoding the Code

We run the string through [Hashes's hash identifier](https://hashes.com/en/tools/hash_identifier). The tool suggests it's Base64. Perfect.

![image.png](/image-15.png)

Using [CyberChef](https://gchq.github.io/CyberChef/), we paste the Base64 string and decode it. We get our first flag:

```
THM{sl1th3ry_tw33tz_4nd_l34ky_r3pl13s!}
```

![image.png](/image-16.png)

## The Sidekick

### Spotting the Sidekick

Back in Task 1, we already encountered a mysterious figure, the one who sent that encoded message to **v3n0mbyt3_**. Their handle? ***[myst1cv1x3n](https://www.threads.com/@_myst1cv1x3n_)***. This was our prime suspect for the role of sidekick.

We began by searching for ***myst1cv1x3n*** across social media platforms. Soon enough, [their Instagram account](https://www.instagram.com/_myst1cv1x3n_) surfaced. Most of the posts were ordinary, but one stood out: [a link to their **SoundCloud** profile](https://soundcloud.com/v1x3n-195859753).

![image.png](/image-14.png)

### The Accidental Leak

On SoundCloud, we discovered something unexpected. Among the tracks, there was [an audio file](https://soundcloud.com/v1x3n-195859753/prototype2) where ***myst1cv1x3n*** casually talked about their operation. But the real treasure was hidden in the description.

```
VEhNe3MwY20xbnRfMDBwczNjX2Yxbmczcl9tMXNjbDFja30=
```

![image.png](/image-19.png)

### Decoding the Message

Just like before, we suspected this was an encoded message. Running it through a Base64 decoder confirmed our hunch. Using CyberChef, we revealed the second hidden flag:

```
THM{s0cm1nt_00ps3c_f1ng3r_m1scl1ck}
```

## The Last Operator

With the leader and sidekick exposed, the hunt for the final member of the **Sneaky Viper group** was on. This time, the trail was subtler, but every operation has weak links, and we were determined to find them.

### Back to SoundCloud

We returned to ***myst1cv1x3n***’s SoundCloud account. Digging deeper, we examined their followers list. Most seemed ordinary, but one stood out: **sh4d0wF4NG**. The name fit the group’s pattern perfectly.

Unlike the others, **sh4d0wF4NG** had no Instagram, Threads, or Twitter presence. But there was one clue: they had liked the audio track where the operators discussed scripts and code. That was enough to push us toward a different angle. Developers often leave traces elsewhere.

Sure enough, a search revealed that **sh4d0wF4NG** maintained a **GitHub** account. On their profile, only one repository appeared: **"red-team-infra."** The name alone suggested offensive security tooling, exactly the kind of infrastructure a hacking group might use.

![image.png](/image-20.png)

### The Exposed Commit

Inspecting the commit history, we found something incriminating. [One commit](https://github.com/sh4d0wF4NG/red-team-infra/commit/78de1f17c45b994e97b8629aa7e5f42c31a0e7f7) added an **IAM AWS user profile**, but the real mistake was in the details: an exposed password, encoded in Base64, sitting right there in the commit.

We copied the encoded string and ran it through CyberChef. Once decoded, the final flag emerged:

```
THM{sh4rp_f4ngz_l34k3d_bl00dy_pw}
```

![image.png](/image-21.png)

&nbsp;