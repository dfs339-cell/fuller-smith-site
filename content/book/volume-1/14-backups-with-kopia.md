---
title: "Chapter 14: Backups with Kopia"
weight: 14.0
draft: false
---

## The Chapter I Wasn't Supposed to Be Able to Write Yet

When I first sketched the outline for this book, I wrote myself a note against this exact chapter: test a restore before you write it. That instruction wasn't decoration. A backup strategy you've never actually restored from isn't a backup strategy — it's a theory, resting entirely on the assumption that every piece of it will work correctly on the one day you actually need it to, untested, under pressure, possibly with something already having gone wrong elsewhere in your stack at the same time.

I'm writing this chapter without having done that test. I want to say that plainly, in the first section, rather than let it surface as a footnote later — because unlike the nginx gap in Chapter 11 or the Portainer version in Chapter 13, this isn't a "haven't gotten round to it yet, no real urgency" gap. This is the one gap in this entire book that's actively dangerous right now, today, regardless of whether anything ever goes wrong. Every day this backup strategy runs untested is a day I'm trusting something I have no actual evidence works.

I think that's worth more to you as a reader than a chapter pretending this was already sorted before I sat down to write it. So here's what I actually have running, why it's probably not enough as configured, and exactly what the test I'm about to go do looks like — written so you can do it properly, in a way I haven't yet.

## What's Actually Running

Kopia is doing scheduled backups every 12 hours, retaining the 10 most recent snapshots, stored on a separate D: drive rather than the same disk as the data being backed up — that separation is the one piece of this I'm confident is right, since a backup living on the same failing drive as the thing it's backing up isn't really a backup at all.

Setting this up, if you're starting from scratch, looks like:

```powershell
kopia repository create filesystem --path=D:\kopia-backups
kopia policy set --global --keep-latest=10
```

And a scheduled task, run via Windows Task Scheduler every 12 hours, executing:

```powershell
kopia snapshot create C:\path\to\your\stack
```

That's the mechanical setup, and it's genuinely not complicated — Kopia handles deduplication and incremental snapshots well, so 12-hourly backups aren't as storage-hungry as that cadence might sound.

## The Retention Window Problem

Twelve-hourly snapshots, ten retained, works out to five days of history. I flagged this to myself as "may be shallower than intended" before I'd even gotten to writing this chapter, and sitting with it properly now, I think that flag was right. Five days covers you against a mistake you notice quickly — an accidental deletion, a bad config change you catch the same day. It does not cover you against something you don't notice for a week or two, which is a genuinely common failure pattern: a problem introduced quietly, not causing visible issues immediately, discovered only once it's already outside your entire retention window.

A more properly designed retention policy — the kind I should have set up from the start rather than the flat "keep the last ten" I actually have — layers different retention periods at different granularities: recent snapshots kept frequently, older ones thinned out but kept for longer. Kopia supports exactly this natively:

```powershell
kopia policy set --global --keep-hourly=24 --keep-daily=14 --keep-weekly=8 --keep-monthly=6
```

That configuration keeps the last 24 hourly snapshots, 14 daily ones beyond that, 8 weekly beyond that, and 6 monthly beyond that — meaningfully deeper coverage than a flat five-day window, at a storage cost that's still reasonable because older snapshots thin out rather than accumulating at full density forever. This is the policy I'm switching to, not the one I've actually been running while writing the rest of this book.

## Why an Untested Backup Isn't Actually a Backup

Here's the reasoning I'd ask you to sit with, because it's the actual point of this chapter, more than any specific Kopia command: a backup can fail silently in ways that only reveal themselves at restore time. A scheduled task that stopped running weeks ago without you noticing. A repository that's been quietly corrupting. Permissions that changed and silently broke the backup job while still reporting success. None of these show up by looking at a snapshot list that says "yes, backups exist" — they only show up when you actually try to bring something back and discover it doesn't work, which is exactly the worst possible moment to discover it.

This is why the instruction I wrote myself — test a restore before writing this chapter — mattered in the first place, and why skipping it isn't a minor shortcut. A tested restore is the only thing that actually converts "I have backups" from a hopeful assumption into a verified fact.

## The Test I'm About to Go Run

Since I couldn't hand you results I didn't have yet at the time, here's exactly what I set out to do the moment this chapter was drafted, written clearly enough that you can run the identical test yourself rather than waiting on me to report back.

**Pick a real, non-trivial piece of the stack to restore** — not a single small file, which proves too little, but something substantial enough that a partial or corrupted restore would actually be obvious. A full model directory, or the entire Open WebUI data volume, is a reasonable choice.

**Restore it to a different location, not back over the original** — this is important discipline, not paranoia: restoring on top of working data means if the restore itself is subtly broken, you've just damaged your working copy to prove it.

```powershell
kopia snapshot restore <snapshot-id> D:\restore-test
```

**Actually verify the restored data, don't just confirm files appeared.** Open a restored model file and confirm it loads correctly in Lemonade. Open a restored Open WebUI database and confirm chat history is genuinely intact, not truncated or corrupted. This is the same "read the response, don't just confirm one arrived" discipline this book has leaned on since Chapter 4, applied here to backups instead of model responses.

**Time how long it actually took.** A restore that works but takes six hours changes your real disaster-recovery expectations in a way worth knowing about calmly, in a test, rather than discovering under pressure during an actual incident.

## What I'd Ask of You, Specifically

Don't let this chapter be the second one in this book — after nginx in Chapter 11 — where I've told you honestly what's unfinished and left it there. Run the restore test before you consider your own backup strategy real, regardless of whether I've reported back on mine by the time you're reading this. A five-minute test now is a small enough cost that there's genuinely no good reason to defer it the way I have, and I'd rather you not make the same mistake I'm actively correcting in this chapter, even as I write it before actually doing the thing myself.

## A Postscript: The Test I Didn't Plan to Run

I hadn't yet done the deliberate restore test described above — the full model directory, the checklist, the pass/fail I promised myself — by the time I got to writing this postscript. But something close to it happened anyway, unplanned, while building the research pipeline covered in Chapter 17. A script bug caused a real file to get overwritten mid-session — genuine data loss, not a drill. Kopia's 12-hourly snapshot cadence happened to have captured a version from shortly before the overwrite, and a targeted restore of just that one file, pulled out to a separate location and checked before trusting it, brought the lost content back intact.

I want to be precise about what this does and doesn't prove, rather than let it stand in for the real test I still owe myself. It confirms the mechanism works — snapshots exist, a specific file can be pulled out of one, and the restored content is genuinely correct rather than corrupted. It does not confirm what the planned test above is actually designed to check: a full, non-trivial directory restored to a clean location and verified end to end, the scenario that would actually matter in a real disaster rather than a single-file mistake caught within the same day it happened. A single lucky recovery is evidence, not proof, and the gap between those two is exactly the gap this chapter has been honest about from the start.

Chapter 15 moves to knowledge management — setting up Obsidian as a local vault and connecting it into Open WebUI so this whole system can draw on your own notes and documents, not just the open web.
