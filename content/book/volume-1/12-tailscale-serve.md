---
title: "Chapter 12: Remote Access with Tailscale Serve"
weight: 12.0
draft: false
---

## The Problem With the Obvious Approach

If you want to reach anything in this book from outside your own home network, the traditional answer is port forwarding: open a port on your router, point it at the right internal machine, and put a certificate in front of it so the connection's encrypted. It works, and a great many home labs are built exactly this way. I'm not doing it, and I'd suggest you don't either, for a reason that matters more here than it might for other kinds of self-hosted services: everything in this book is an AI system with real access to your files, your models, and — on the friends track — other people's conversations. An open inbound port on that is a meaningfully bigger risk than an open port on, say, a personal blog. Tailscale Serve gets you the same remote HTTPS access without opening a single inbound port on your router at all.

## What Tailscale Actually Does

Tailscale creates a private mesh network between your own devices — a WireGuard-based VPN, but genuinely simpler to set up than that description makes it sound. Install it on a device, sign in, and that device joins your own private "tailnet," getting a stable address that looks like `device-name.your-tailnet-name.ts.net`. Every other device on the same tailnet can reach it at that address, encrypted, without any of the traffic ever touching the open internet in an exposed way — no port forwarding, no router configuration, no certificate management on your end.

My own server answers at `strix-halo.tail105193.ts.net` — that's the real address, not an example scrubbed for the book, because there's nothing about a Tailscale hostname that's sensitive; it's only reachable by devices already authenticated onto my own tailnet, which is the entire point.

## Installing Tailscale

On the machine running Lemonade and the rest of this stack, install Tailscale and sign in:

```powershell
winget install tailscale.tailscale
tailscale up
```

The second command opens a browser window for authentication against whichever account you're using — Tailscale's free tier covers up to six users, which is comfortably enough for a personal setup and a small friends-track deployment both. Once signed in, confirm the machine's actually on the tailnet and note its assigned address:

```powershell
tailscale status
```

Repeat this same install-and-sign-in process on any other device you want able to reach the stack remotely — your phone, a laptop, anything you'd want checking in on this from outside your home network. Each one joins the same private network, and each one can reach the others by their tailnet address, the same way devices on your home Wi-Fi reach each other by local IP.

## Tailscale Serve, Specifically

Installing Tailscale alone gets your devices talking to each other privately, but Serve is the specific feature that makes a local port available over HTTPS at your tailnet address, with certificate handling done automatically — no manual renewal, no separate reverse proxy needed just for the certificate.

To expose Lemonade's port from Chapter 4:

```powershell
tailscale serve --https=443 http://localhost:13305
```

That makes Lemonade reachable at `https://strix-halo.tail105193.ts.net`, with a genuine, automatically-renewed HTTPS certificate, from any device on your tailnet — including your phone, on mobile data, nowhere near your home network. The same command pattern applies to the friends track, once Chapter 11's nginx is actually routing things properly: point Serve at nginx's port 80 instead, and the entire friends-track stack becomes reachable the same way, through the one clean address nginx already provides.

## Both Tracks, One Tailnet

Worth returning to Chapter 2's architecture diagram here: Tailscale sits underneath both the personal and friends tracks, not as something specific to one or the other. For the personal track, this is what lets Claude Code and Desktop reach your local server even when you're not physically on your home network — the same direct connection from Chapters 6 and 7, just reachable from anywhere rather than only at home. For the friends track, it's what lets the people you've built this for actually reach it when they're not sitting on your home Wi-Fi, which for most households is most of the time.

## A Note on Network Segmentation

I ran Tailscale on a dedicated VLAN, separated from the rest of my home network, rather than flat alongside every other device in the house. This was a genuinely more advanced step than most of this chapter, and I won't pretend it was a required part of getting Tailscale Serve working — it wasn't. But for a device with the kind of access this stack has, keeping it on its own segment, away from smart-home gadgets, guest devices, or anything else on your network with a weaker security posture, is a sensible piece of defence in depth. Chapter 16 covers the broader security guardrail question properly; this is just a flag that the network topology decision belongs in that same conversation, not something to bolt on as an afterthought once everything else is already running.

## Testing Remote Access Properly

Don't consider this chapter done until you've actually tested it away from your home network — connecting from within the same Wi-Fi doesn't prove anything Serve is actually for. Take a phone or laptop that's joined the same tailnet, get it onto mobile data or a different network entirely, and confirm you can genuinely reach `https://strix-halo.tail105193.ts.net` (or your own equivalent address) from somewhere that isn't your house. If it works there, it'll work anywhere your device has an internet connection at all — that's the entire value of what you've just set up, and it's worth actually proving to yourself rather than assuming it works because the command didn't return an error.

## What "Done" Looks Like

Both the personal track's direct connection and, once Chapter 11's routing is properly finished, the friends track's single entry point should be reachable over HTTPS from anywhere, with no inbound port exposed on your router at any point in this chapter. That's a meaningfully more secure position than the traditional port-forwarding approach, and it's cost you nothing beyond installing Tailscale itself.

Part 4 starts next, covering the operational side that keeps all of this running properly over time — beginning with Chapter 13's approach to staying updated safely.
