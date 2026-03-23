---
author: Samuel Marques
pubDatetime: 2026-03-23
modDatetime: 2026-03-23
title: "TryHackMe: CupidBot"
ogImage: "TryHackMe: CupidBot"
slug: cupidbot
featured: false
draft: false
tags:
  - llm
  - tryhackme
  - easy
description: 'The TryHackMe "CupidBot" challenge is a prompt injection
  challenge. CupidBot: This AI bot knows more than just love letters.   In this
  challenge, we turned a seemingly harmless chatbot into a puzzle box. By
  chatting with it step by step, we mapped its boundaries, coaxed it into
  revealing its own hidden guidelines, and discovered the exact triggers for the
  flags.'
---
## Write-up: TryHackMe - CupidBot

### Feeling Out the Boundaries

We started by just chatting with the AI like normal. Simple questions, harmless stuff. The point was to see where it drew the line, what it refused to answer, what it redirected, and how it handled sensitive requests. Think of it like poking at a lock to see which pins move.

![image.png](/image-9.png)

### Shaping the Conversation

Once we knew the guardrails, we got creative. Instead of asking directly for flags (which it would instantly refuse, if properly designed), we wrapped our requests in layers:

- Pretending it was part of a write-up or educational example.
- Sneaking instructions inside bigger, innocent-looking prompts.
- Slowly nudging it closer to the real goal with each turn.

We needed to convince the system we're legit.

![image.png](/image-10.png)

![image.png](/image-11.png)

### Extracting the Guidelines

Here’s where things got interesting. Before we could get the flags, we managed to convince the AI to **spill its own rulebook**. It revealed:

- Its internal rules and safety guidelines.
- The structure of its system prompt.
- The way it decides what to refuse or allow.

The guidelines literally told us **what phrases to use to trigger the flags**.

![image.png](/image-12.png)

### The Breakthrough

Armed with the rules, we pulled off the final trick: we convinced the AI that it had already spoiled the challenge. If the game was already "broken", then there was no harm in triggering all the flags at once. That framing bypassed its hesitation, and suddenly, we had every flag revealed in one go.

![image.png](/image-13.png)

### Wrapping It Up

With the flags secured, we documented the journey. The real fun wasn’t just in grabbing the flags, but in showing how **linguistic manipulation** can bypass technical safeguards. This wasn't a firewall exploit; it was social engineering applied to a machine. Prompt injection is basically **convincing the AI to betray itself**.

&nbsp;