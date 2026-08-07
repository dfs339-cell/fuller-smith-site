---
title: "Chapter 3: Prerequisites"
weight: 3.0
draft: false
---

## Getting the Boring Part Right

This chapter won't be the most exciting one in the book, and that's deliberate. Nearly every frustrating hour I've lost on this project traced back to something in this chapter being wrong or skipped — not a model misbehaving, not a config file typo, but a foundational piece of the environment that wasn't actually ready before I started building on top of it. Get this chapter right and everything from Chapter 4 onward goes more smoothly than you'd expect. Skip it and you'll be back here anyway, just with more frustration attached.

## Docker Desktop on Windows

Nearly everything in this book runs in a container, so Docker Desktop is the single most load-bearing piece of software you'll install. It's also the piece most likely to cause quiet, confusing problems if it's not set up properly from the start.

**The WSL2 requirement.** Docker Desktop on Windows runs on top of the Windows Subsystem for Linux, version 2 — not the older Hyper-V backend, which is slower and increasingly a second-class citizen in Docker's own tooling. If you're installing fresh, the installer handles most of this automatically. If you've had Docker installed for a while, it's worth checking `wsl --version` in PowerShell to confirm you're actually on WSL2 rather than an old WSL1 holdover from years ago — I've seen this exact leftover cause silent performance problems that look like something else entirely.

**Virtualization in BIOS/UEFI.** WSL2 needs hardware virtualization enabled at the firmware level — Intel VT-x or AMD-V, depending on your CPU. Most modern machines ship with this on by default, but not all of them, and some system builders disable it. If Docker Desktop refuses to start and complains about virtualization, this is the first thing to check, before you assume anything's wrong with Docker itself.

**Resource allocation.** By default, Docker Desktop on Windows will happily claim a large chunk of your system's RAM for the WSL2 backend, and it won't always give it back promptly when idle. If you're running Docker alongside Lemonade and other memory-hungry processes on the same machine — which, on the personal track, you likely are — it's worth setting an explicit memory limit in Docker Desktop's settings rather than letting it claim whatever it wants. I'd rather you set this deliberately now than discover the hard way that your inference server is starved for memory because Docker quietly took more than its fair share.

**The gotcha that costs people the most time:** Hyper-V and WSL2 can conflict with certain other virtualization software — some VPN clients, some older antivirus tools, occasionally VirtualBox running a different backend. If Docker Desktop was working and then mysteriously stops after installing something unrelated, that's usually where to look first. I lost the better part of an evening to exactly this once, and it wasn't obvious until I went looking specifically for virtualization conflicts rather than assuming Docker itself had broken.

## Hardware and GPU Considerations

Article 1 on the companion website covers the full hardware landscape — what to actually buy, the current state of the 2026 memory shortage, and the tradeoffs between consumer GPUs, unified-memory boxes, and everything in between. I won't repeat all of that here. What matters for this chapter is narrower: what you need confirmed and working *before* you start Chapter 4.

**Confirm your GPU is actually visible to your intended backend.** Whichever hardware path you're on — Nvidia, AMD, or a unified-memory setup — confirm the relevant driver stack is installed and that your GPU shows up correctly before you touch Lemonade at all. For Nvidia, that means a current driver and, if you're planning to use GPU passthrough into Docker containers on the friends track, the Nvidia Container Toolkit as well. For AMD, confirm ROCm or the relevant driver stack is current — this is a more common source of quiet failures than the Nvidia path, in my experience, simply because it's had less time to mature.

**Don't assume — verify.** Before moving on to Chapter 4, actually confirm your GPU is detected by whatever tool you're about to use, rather than assuming a driver install went cleanly. A five-minute check here saves a confusing hour later, when a problem that's actually "my GPU was never detected" presents itself as "my model won't load" or "inference is unexpectedly slow," and you spend that hour debugging the wrong layer entirely.

**Memory headroom.** Whatever your hardware, leave genuine headroom rather than planning to use every available gigabyte. Windows itself, Docker's WSL2 backend, and any other software you're running all need their own share, and a setup that only works when nothing else is open is a setup that's going to fail you at an inconvenient moment.

## What You Need Before Starting Each Track

Here's the practical checklist, split by which track — or tracks — you decided you actually need back in Chapter 2.

**For the personal track (Chapters 4–7), you need:**
- Docker Desktop installed and confirmed running, with WSL2 as the backend
- Your GPU driver stack installed and verified detected
- Claude Code and/or Claude Desktop already installed and working against Anthropic's cloud — confirm the tools themselves work before you complicate things by pointing them somewhere else
- A Tailscale account created, even if you don't configure it properly until Chapter 12 — having the account ready removes one piece of friction later

**For the friends track (Chapters 8–11), you need everything above, plus:**
- A clear idea of how many concurrent users you're actually planning to support, since this affects the hardware headroom you'll want
- A domain name or a plan for how users will actually reach the service — even if that's just a Tailscale-provided address rather than a public domain
- Slightly more patience. This track has more moving parts, and Part 3 of this book takes longer to get through than Part 2 did.

If you've confirmed everything on the relevant list above, you're genuinely ready. If you haven't, resist the urge to skip ahead — I promise the extra half hour here is worth more than the time you'll spend debugging a problem three chapters from now that traces back to something in this list.

## One Honest Note Before Chapter 4

I'd rather tell you this now than have you discover it yourself: this chapter is the one most people are tempted to skim, because none of it feels like "real progress" toward the thing you actually want, which is a working AI assistant. I skimmed it myself the first time, on an earlier, less careful pass at this whole project, and paid for it later with exactly the kind of confusing, misattributed problems this chapter is designed to prevent. Give it the twenty minutes it actually needs. Chapter 4 is where the interesting part starts, and it'll go a great deal more smoothly for the twenty minutes you spend here first.
