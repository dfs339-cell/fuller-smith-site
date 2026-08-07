---
title: "Chapter 2: Architecture Overview"
weight: 2.0
draft: false
---

## One Stack, Two Very Different Jobs

Before you install a single piece of software, it's worth understanding the decision that shapes everything else in this book: you're not really building one thing. You're building a piece of shared infrastructure — local AI inference — and then deciding how many different ways you want to expose it.

I learned this the slow way, by building it wrong first. My early setup treated "give myself access" and "give other people access" as the same problem with the same solution, and it wasn't. A setup built for one trusted, technical user behaves completely differently from a setup built for several non-technical people who just want something that works when they open an app. Trying to serve both with one configuration meant I was either over-engineering my own daily use with infrastructure I didn't need, or under-serving everyone else with something too fragile for someone who doesn't want to think about how it works. Once I split it into two deliberately different tracks, both problems went away at once.

That's the architecture this whole book is built around, so it's worth getting properly clear on before Chapter 3 has you installing anything.

## The Personal Track

The personal track is built for exactly one kind of user: you, technical, already comfortable with a terminal or a desktop app, wanting a private, capable AI assistant that behaves like Claude but runs entirely on your own hardware.

The tools here are Claude Code and Claude Desktop, pointed at your own local inference server instead of Anthropic's cloud. Both of these already speak the Anthropic API natively — that's the entire trick. If your local server can be made to answer in the same shape Anthropic's cloud does, neither tool needs to know or care that it's talking to a box on your own network rather than a data centre. Chapters 4 through 7 walk through exactly how that gets wired up.

The defining characteristic of this track, and the thing that makes it genuinely simple compared to what's coming in Part 3: it's a direct, single-user, trusted connection. Your machine talks to your server, over your own network — I use Tailscale for this, which Chapter 12 covers properly — and there's no need for a reverse proxy, no need for authentication routing, no need for anything designed to manage multiple simultaneous users with different permission levels. It's you, talking to your own hardware, the same way you'd open a file on your own drive. That simplicity isn't a limitation. It's the whole point of this track — it's meant to be the low-maintenance, high-trust option, and every extra piece of infrastructure you might be tempted to add to it is something you'll be maintaining for no real benefit.

## The Friends Track

The friends track solves a completely different problem: giving other people — family, actual friends, anyone who isn't you and isn't going to open a terminal — access to the same underlying AI capability, through something that looks and feels like using ChatGPT or Claude.ai, no technical knowledge required on their end.

This is built around Open WebUI as the front end, with Vane (or Perplexica, depending on what you prefer) handling web search, and SearXNG behind that as a private search backend rather than paying for a commercial search API. Chapter 8 through 11 cover all of it.

The reason this track needs meaningfully more infrastructure than the personal track isn't complexity for its own sake — it's a direct consequence of the actual requirement. Multiple users means you need routing that can direct different people to the right service invisibly. Non-technical users means the whole thing needs to be genuinely idiot-proof, not "technically works if you know the right URL." And a web-facing service, even one that's only reachable inside your own network or over Tailscale, benefits from nginx sitting in front of it — handling routing cleanly, giving you one sensible place to manage access rather than scattered ad-hoc ports everyone has to remember. None of that is needed for a single trusted user talking directly to their own machine. All of it earns its place the moment you're serving more than one person.

## Why the Split Actually Matters

Here's the architectural principle worth taking away from this chapter, because it'll save you real time later: match the infrastructure to the actual trust and complexity level of who's using it, not to whichever setup looks more impressive.

I've watched this exact mistake made with infrastructure far more expensive than a home AI lab, in an earlier career selling enterprise IT — a business builds one heavyweight, fully-routed, multi-user-ready system because that's what "proper" infrastructure looks like, then makes a single person go through all of it just to check their own email. That's wasted complexity, and wasted complexity isn't neutral — it's more things that can break, more things to patch, more things you have to remember how you configured six months later when something stops working. The personal track in this book is deliberately minimal because a single trusted user doesn't need more than that. The friends track is deliberately more built-out because multiple non-technical users genuinely do. Neither is a compromise. They're both the right amount of infrastructure for the job they're actually doing.

## The Full Stack, at a Glance

Here's the shape of the whole thing, which I'd suggest bookmarking this page for — every chapter from here on slots into one part of this diagram.

```
                        ┌─────────────────────────────┐
                        │   Local Inference Server     │
                        │   (Lemonade — Chapter 4)     │
                        │   Serving your local models  │
                        └───────────────┬───────────────┘
                                        │
                    ┌───────────────────┴───────────────────┐
                    │                                       │
        ┌───────────▼───────────┐               ┌───────────▼───────────┐
        │   PERSONAL TRACK       │               │   FRIENDS TRACK        │
        │   (Part 2)             │               │   (Part 3)             │
        │                        │               │                        │
        │  Claude Code /         │               │  Open WebUI            │
        │  Claude Desktop        │               │       │                │
        │       │                │               │  Vane / Perplexica     │
        │  Model aliasing        │               │       │                │
        │  (Ch.5) + Gateway      │               │  SearXNG (search)      │
        │  config (Ch.6)         │               │       │                │
        │       │                │               │  nginx (routing)       │
        │  Direct connection —   │               │       │                │
        │  no router needed      │               │  Multi-user access     │
        │                        │               │                        │
        └───────────┬───────────┘               └───────────┬───────────┘
                    │                                       │
                    └───────────────────┬───────────────────┘
                                        │
                        ┌───────────────▼───────────────┐
                        │   Tailscale (Ch.12)             │
                        │   Remote access, both tracks    │
                        └─────────────────────────────────┘
                                        │
                        ┌───────────────▼───────────────┐
                        │   Part 4: Operations            │
                        │   Updates, backups, security,   │
                        │   knowledge management —        │
                        │   shared across both tracks      │
                        └─────────────────────────────────┘
```

Both tracks share the same underlying inference server — you're not running two separate AI stacks, just two different front doors onto the same one. And everything in Part 4 — staying updated, backups, security, knowledge management — applies regardless of which track or tracks you've built, which is why it's covered once, at the end, rather than duplicated for each.

## Deciding What You Actually Need

Before Chapter 3 has you installing Docker Desktop, it's worth being honest with yourself about which track you actually need, because building both when you only need one is exactly the wasted-complexity mistake this chapter just warned you about.

If it's just you, and you're comfortable with a terminal or a desktop app, the personal track alone is a complete, satisfying project — you can stop after Chapter 7 and have a fully working private AI assistant, with the rest of the book as optional depth. If you've got family or friends who'd genuinely use this too, but you don't want to be their tech support for a command line, the friends track is worth the extra setup. And if you want both — your own low-friction daily driver, plus something you can point people you like toward — that's exactly how I've built mine, and it's why this book covers both properly rather than picking one.

There's no wrong answer here. There's only the answer that matches who's actually going to use this once it's built.

Chapter 3 gets your environment ready for whichever path you've chosen. Let's get into it.
