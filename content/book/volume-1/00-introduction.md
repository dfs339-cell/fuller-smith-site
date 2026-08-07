---
title: "Introduction"
weight: 0.0
draft: false
---

## The Bill That Started This

I didn't set out to write a book about self-hosting AI. I set out to look properly at what I was actually spending, because the number had started to bother me.

I was using AI heavily for day-to-day work — genuinely useful, genuinely part of how I got things done — and the token spend was climbing in a way that was easy to justify month to month and much harder to justify looking at the trend line. At the same time, the wider market was moving in a direction that made the trend line worse, not better: more of the industry shifting toward per-token billing, more of the cost tied directly to how much you actually used the thing that had just become indispensable. That's a reasonable way for a provider to price a service. It's a less reasonable position for me to be in indefinitely, watching a cost scale linearly with exactly the behaviour I had no intention of reducing.

So I did the maths, the way you do when a subscription number starts nagging at you. And somewhere in doing that maths, a second, quieter question surfaced alongside the cost one: what actually happens to everything I've been typing into these chat windows. Client details. Half-formed ideas. Genuine business thinking, typed out in full because that's the only way to get a useful answer back. I realised I had no real way to answer that question, and that bothered me more than the invoice did.

Those two things together — a cost curve heading the wrong way, and a privacy question I couldn't actually answer — are what made local AI look less like a hobbyist curiosity and more like a genuine edge. Not free, not effortless, but a real alternative with a fundamentally different cost shape: the marginal cost of another query drops to nothing once the hardware's paid for, and nothing you type leaves your own network unless you decide it should.

## Choosing the Box: GMKtec, a Stroke of Luck, and the Bosgame M5

Committing to spend real money on a Strix Halo box didn't mean committing to a specific one straight away. There's more than one manufacturer building on the same underlying chip, and I spent a genuine stretch of time reviewing the options before landing on anything.

The Minisforum S1 was a prime example of what else was out there — a perfectly credible box, built on the same chip, sitting in the same rough category as everything else I was comparing. Framework was another, and genuinely tempting on reputation alone, but their pricing was sky high relative to the rest of the field, enough that it ruled itself out fairly early rather than needing much deliberation.

GMKtec was the one I was most seriously looking at first — they had offers running at the time, and on paper it looked like the obvious choice. I was specifically eyeing their 96GB configuration, more unified memory than I strictly needed at the time but the kind of headroom that's hard to say no to once you've seen the number. It was out of stock.

That stockout is the actual hinge of this whole story, and worth being direct about rather than just calling it lucky in passing: had it been available, I'd have bought 96GB and stopped looking, genuinely happy with the decision at the time. Instead I kept comparing, found the Bosgame M5, and ended up with 128GB — more memory than the GMKtec I'd been about to commit to, at a lower price than I'd have paid for the smaller box. Missing out on the GMKtec wasn't a near-miss I got lucky recovering from. It's the reason I ended up with meaningfully better hardware for meaningfully less money, and I don't think I'd have found that path if the easier option had simply been available when I went looking for it.

Being forced to keep looking is how I came across the Bosgame M5 — 128GB of unified memory and a 2TB NVMe drive, meaningfully cheaper than any of the other Strix Halo boxes I'd been comparing, GMKtec and Minisforum included, for what looked like a comparable spec. That price gap was enough to make me stop and actually dig into why, rather than assume "cheaper" simply meant "worse" and move on. Whatever the answer to that turned out to be, it was cheap enough, relative to the rest of the field, that it changed the shape of the whole decision — the £2,100 I ended up spending was meaningfully less than I'd been resigned to paying a GMKtec-shaped price for, for a box doing the same underlying job.

I'll add one honest regret to this, since the rest of this book doesn't shy away from them: had I bought at launch rather than waiting and comparing the way I did, I'd likely have paid nearer £1,500 for the same category of hardware. That's not a small gap, and it's worth naming plainly rather than letting the £2,100 figure stand as if it were always the going rate. The Strix Halo category moved the way a lot of new hardware categories do — early pricing settles, competition arrives, and the person who waits pays a premium for the comparison-shopping rather than getting rewarded for it. I don't think that makes the decision wrong. It does mean the number in this book is honestly what I paid, not necessarily what the hardware was worth at any given moment.

## The £2,100 Decision

I want to be honest about the number, because I think the honesty is more useful to you than a vaguer version of this story. The Bosgame M5 landed at £2,100 — real money, committed before I knew for certain the whole thing would work the way I hoped, on a category of hardware I hadn't bought before, for a use case I was still partly guessing at the shape of.

That purchase is where this book actually starts, more than any of the technical chapters that follow. Everything from Chapter 1 onward is what happened after that decision — what I built, what broke, what I'd do differently. But the decision itself was the real turning point: the moment the token-spend math and the privacy question stopped being background noise and became a specific, committed, slightly nerve-wracking purchase.

## The First Two Weeks

I want to be honest about this part too, rather than let the story jump straight from "bought the box" to "here's the finished build," because that jump would be dishonest about how it actually felt at the time. The first two weeks were genuinely hard. Not hard in the satisfying, learning-something-new way — hard in the way where I sat in front of the machine more than once, well outside my normal comfort zone, wondering seriously whether I was ever actually going to crack this.

I'd spent £2,100 on hardware I was still learning to understand, on a category of problem — hardware detection, backend selection, the whole unfamiliar stack of concepts that Chapter 3 eventually distills into a checklist — that I hadn't lived with before. None of it was intuitive yet. Docker, WSL2, driver stacks, backend selection: each individually explicable, but stacked together, in those first two weeks, it didn't feel like a system I was learning. It felt like a system that was quietly beating me, one unfamiliar error at a time.

I'm including this not to be dramatic about it, but because I think it's the honest missing piece in most guides like this one — the tidy version skips straight past the part where the person writing it genuinely doubted themselves, and that skip does readers a disservice. If you hit the same wall in your first two weeks that I did, that's not a sign you've done something wrong or that you're less capable than whoever wrote the book you're holding. It's apparently just what this particular learning curve feels like from the inside, before enough of the pieces click into place that it starts feeling like a system you actually understand rather than one that's fighting you.

It did click, eventually — that's the rest of this book. But it didn't click in week one, and I'd rather you know that going in than discover it yourself and wonder if something's uniquely wrong with your own attempt.

## What This Book Actually Documents

This book is the honest record of what came after that purchase — a working, tested build, not a theoretical one, with the failures left in rather than quietly edited out. If you're weighing the same two things I was — a cost curve that only moves one direction under per-token pricing, and a privacy question you can't quite answer about where your own words actually go — the chapters that follow are the practical version of the decision I made, written so you don't have to guess your way through it the way I did.

Chapter 1 picks up the case for self-hosting properly. Let's get into it.
