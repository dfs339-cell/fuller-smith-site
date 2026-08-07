---
title: "Chapter 17: The Vault That Feeds Itself"
weight: 17.0
draft: false
---

## From Something You Fill to Something That Fills Itself

Chapter 15 set up Obsidian as a place to keep your own notes, and Chapter 16 locked that vault down. Both chapters assumed a person doing the filling — you, dropping in a document, writing a note, pasting something worth keeping. That's a fine place to stop. Most people reading this book probably will, and there's nothing wrong with a vault that only ever grows as fast as you personally feed it.

This chapter is for the version of this project I actually wanted: a vault that keeps growing on its own, overnight, whether or not I sat down and added anything myself. Not a chatbot that answers questions about what I already know — a system that goes out, finds things I didn't know to look for, and has them waiting for me the next time I open Obsidian. That's a genuinely different capability from anything earlier in this book, and it's worth its own chapter rather than a footnote on Chapter 15.

## The Shape of It

Six scripts, each responsible for one source, each writing into its own corner of the vault:

- **ArXiv** — recent papers matching a keyword list, queried against the official API
- **GitHub** — trending repositories on topics you actually care about
- **Web/News** — a broader sweep across a self-hosted search backend, a news feed, and a couple of blogging-platform tag feeds
- **YouTube** — transcripts of recent videos matching your topics, summarized into structured notes
- **UniFi** — the same YouTube approach, aimed at a completely different subject
- **Reddit** — in principle, top posts from a handful of subreddits. In practice, the next section explains why this one doesn't currently work at all.

All six run unattended, on a schedule, through Windows Task Scheduler — the same tool Chapter 14 already had you using for Kopia's backup cadence. Nothing here needs a new piece of infrastructure. It needs the infrastructure you already have, pointed at a new job.

## Why Not Just One Script

The obvious question, if you're building this yourself: why six separate scripts instead of one script with six functions? Two reasons, both learned rather than assumed going in.

The first is failure isolation. If GitHub's API starts rate-limiting you, that failure shouldn't take ArXiv down with it. Six independent scripts mean six independent failure domains — one source having a bad night doesn't touch the other five.

The second is pacing. Each of these sources has its own rhythm — different rate limits, different response times, different appropriate delays between requests. A single monolithic script trying to satisfy all six at once either over-throttles the fast ones or under-throttles the slow ones. Separate scripts, separate schedules, each tuned to what its own source actually needs.

## A Worked Example: The ArXiv Script

The shape is similar across all six, so one worked example carries most of the pattern. Stripped to its core:

```powershell
$Keywords = @("quantization", "mixture of experts", "efficient inference")
$SeenFile = "C:\scripts\state\arxiv-seen.json"

$seen = @{}
if (Test-Path $SeenFile) {
    $seenData = Get-Content $SeenFile -Raw | ConvertFrom-Json
    foreach ($item in $seenData) { $seen[$item.Id] = $item.DateAdded }
}

$newPapers = @($allPapers | Where-Object { -not $seen.ContainsKey($_.AbsUrl) })

if ($newPapers.Count -eq 0) {
    Write-Log "Nothing new. Exiting."
    exit 0
}

# ... write $newPapers to the vault, then update $seen and save it back
```

Two things worth calling out, both because they're the kind of detail that's easy to skip past and expensive to get wrong.

**The `@()` around that `Where-Object` isn't decoration.** In Windows PowerShell 5.1, a pipeline that matches zero results collapses to `$null` rather than an empty array, and a pipeline that matches exactly one result collapses to a bare object rather than a one-item array. Either way, `.Count` on the result becomes unreliable — and the specific failure mode is nasty: a script can silently skip its own "nothing new, exit early" check, fall through, and overwrite a good file with an empty one. This happened to me once, for real, and it's why every dedup check in this pipeline wraps its `Where-Object` in `@()` now, not just the ones where I noticed a problem.

**The seen-list is what makes this safe to re-run.** ArXiv's API has no concept of "give me only what's new since last time" — every query returns everything matching, regardless of whether you've seen it before. Without something tracking what's already been written to the vault, running this script twice in one day would write every paper twice. The seen-list, one JSON file per source, is the entire mechanism that turns a stateless API into something that behaves like an incremental feed.

## Reddit: A Case Study in an API That Just Stopped Existing

I said above that Reddit "in principle" works the same way as the other five, and I want to walk through why it doesn't, because it's a genuinely useful thing to know if you're building anything against a third-party API in 2026 rather than a specific lesson about Reddit alone.

For years, appending `.json` to any Reddit URL returned clean, structured data with no authentication required — a well-known, widely-documented trick, and the foundation the Reddit script in this pipeline was originally built on. In November 2025, Reddit closed self-service API registration entirely, replacing it with a manual approval process. That alone didn't break anything already running — existing integrations kept working. Then, on May 30th 2026, Reddit went further and killed the unauthenticated `.json` endpoints outright, with no warning and no migration path. The same request that had worked the day before started returning a 403 and a message pointing you at "developer credentials" instead of data.

I tried the obvious fallback — Reddit's `.rss` feeds, which have historically had a looser access policy than `.json`. Blocked too, with an explicit message: *"if you're running a script or application, please register or sign in with your developer credentials."* There's no clever workaround here. This isn't a bot-detection quirk to route around the way Chapter 9's SearXNG limiter was. It's a deliberate policy decision, and the only real path forward is filing a formal access request and waiting on Reddit's review — no guaranteed timeline, no guaranteed approval.

The Reddit script in this pipeline is still scheduled, still runs every night, and correctly logs "nothing new" and exits cleanly rather than erroring or crashing anything else. It's parked, not deleted — if access is ever approved, the discovery-and-dedup structure doesn't need to change, only the authentication layer. But as of this writing, it's the one source in this chapter that doesn't actually produce anything, and I'd rather tell you that plainly than have you build this yourself and assume you'd done something wrong when it doesn't work.

## Task Scheduler as the Backbone

Six scripts need six schedules, staggered enough that none of them contend for Lemonade's attention at once:

```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" `
             -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$scriptPath`""
$trigger = New-ScheduledTaskTrigger -Daily -At $time
$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable `
              -RestartCount 2 -RestartInterval (New-TimeSpan -Minutes 5)

Register-ScheduledTask -TaskName $name -Action $action -Trigger $trigger `
    -Settings $settings -User $env:USERNAME
```

`-ExecutionPolicy Bypass` matters for a reason specific to this setup: every script here gets downloaded rather than typed by hand, which means Windows flags it with a "this came from the internet" zone marker, and a normal execution policy would refuse to run it without an explicit `Unblock-File` first. Bypass sidesteps that entirely for scheduled runs — worth knowing if you ever wonder why a script that needed unblocking to test manually just works once it's scheduled.

`-RestartCount` and `-RestartInterval` are there because a scheduled task that runs once and dies on failure isn't meaningfully more reliable than not scheduling it at all. If a script crashes — a transient network failure, an API having a bad moment — Task Scheduler retries it a couple of times before giving up, rather than silently staying failed until the next day's run.

That said, a boot-only or run-once trigger has a real limitation worth understanding before you rely on it, because I found this one out the expensive way: a trigger that only fires under a specific condition — at boot, say — gives you no protection against a process that dies *after* it starts successfully. If something crashes mid-run and there's no reboot to re-trigger it, it just stays dead, silently, for as long as nothing forces a restart. The fix isn't a different kind of trigger — it's making sure the task's settings include restart-on-failure, not just a start condition, so the task itself notices and recovers rather than depending on something external like a reboot to save it.

## Enrichment: Turning Files Into a Graph

Six scrapers writing markdown files gets you a vault that's fuller, but not smarter — Obsidian will full-text search any of it, but a pile of unconnected files is still just a pile. The piece that actually changes the character of the vault is a separate script, run twice a day, that goes back through recent notes and adds Obsidian's `[[wiki-link]]` syntax around genuine concepts — company names, model names, hardware, techniques — so that notes about the same thing actually connect to each other rather than sitting in adjacent files that happen to mention the same word.

The prompt driving this is deliberately narrow:

```
Identify key concepts, tools, models, and named entities in the text
below, and wrap the FIRST mention of each with [[ConceptName]] syntax.
Do NOT change, reformat, restructure, or reword ANY existing text.
```

That restriction — don't touch anything else, only add links — is the important part. An earlier instinct would have been to let the model "clean up" or restructure notes while it was in there anyway, and that's exactly the kind of scope creep worth resisting: a note that already has good structure doesn't need an LLM reorganizing it, and every additional thing the prompt is allowed to change is one more way it can quietly get something wrong. Narrow the job to the one thing you actually need, and the blast radius of a bad output shrinks with it.

The practical result, after this has been running for a while: open Obsidian's graph view, and instead of isolated dots, you see real clusters — a hardware platform connected to every video, paper, and news article that mentioned it, a model name linking together everything the pipeline found about it across six different sources. That's genuinely different from search, which only ever answers the question you thought to ask. A graph surfaces connections you didn't know to look for.

## Closing the JS-Rendering Gap: Adding Crawl4AI

The six-script shape above held up well, but it had a blind spot I didn't notice until I went looking specifically: the Web/News script does a direct HTTP fetch, and a growing number of pages these days render their actual content client-side, in JavaScript, after the initial page load. A direct fetch against one of those pages gets you nav chrome and boilerplate — nothing resembling the article. Lemonade's own summarisation step is honest about this when it happens; it correctly flags the result as `NO_REAL_CONTENT` rather than confidently summarising nothing. The problem was what happened next: that URL got marked as permanently "seen" the same as any successfully processed one, and was never retried. A genuinely useful source was being silently and irrecoverably lost on the first pass, and the seen-list — the exact mechanism from earlier in this chapter that makes the pipeline safe to re-run — was quietly hiding the loss rather than exposing it.

Before building anything, I looked at the `searxng-crawl4ai-mcp` project on GitHub as a possible drop-in fix. I rejected it: it bundles a full MCP server, Redis, and a second SearXNG instance, and only the scraping/rendering half of that was actually additive to what I already had running. Standing up the rest would have meant duplicating infrastructure Chapter 9 already covers, for no real benefit.

Instead, `unclecode/crawl4ai:latest` runs as its own standalone container — deliberately outside the main Docker Compose stack, in its own `docker-compose.yml` at `C:\scripts\crawl4ai\`, on port `11235`, with no shared network. That isolation is deliberate: it can be rebuilt or restarted without touching Open WebUI, SearXNG, or anything else this book has already gotten working. It reuses the existing SearXNG instance rather than standing up a second one. Two things worth knowing if you're setting this up yourself: the current version defaults to loopback-only binding, so reaching it from a PowerShell script at all means explicitly setting `CRAWL4AI_API_TOKEN`; and content extraction runs through a `PruningContentFilter` at `threshold: 0.6`, which strips navigation, ads, and cookie banners down to the actual article body — the difference between the clean `fit_markdown` field and the noisy `raw_markdown` one.

The pipeline integration is narrow and deliberately conditional. A new function, `Get-Crawl4AIMarkdown`, triggers specifically on the `NO_REAL_CONTENT` branch — only *after* Lemonade itself has already judged a direct fetch to be boilerplate, not on every fetch. That keeps the common case — direct fetches that already work fine, which is most of them — completely untouched, and only pays the extra cost where it's actually needed. On success, the Crawl4AI-fetched content gets run back through the same summarisation logic, `Get-ArticleSummary`, refactored into a function both code paths now share rather than duplicating the prompt. On failure, the script falls through to the original permanent-skip behaviour — no worse off than before Crawl4AI existed, just with a real first attempt inserted ahead of giving up.

## Housekeeping: Legacy Filename Cleanup

A separate discovery, found while working through the vault rather than while building anything new: roughly 120 files in `digests\youtube` were still sitting under an old naming convention — `yt-<date>-<videoID>.md` — with a different frontmatter schema (`id:`, `vertical:`, `status:`, `publish:`, `supersedes:`) than the current pipeline produces.

These trace back to the earlier Claude-Code-CLI-subagent architecture, before the migration to the direct PowerShell-to-Lemonade calls this chapter has been describing — a genuine artifact of the pipeline's own evolution, not a bug worth chasing. A one-time rename script cleaned them up, matching each file to the current title-slug convention using the `title:` field already present in its frontmatter, with no content changes. One filename collision surfaced along the way and was handled correctly by the script's dedup-suffix logic; it turned out to be a likely duplicate capture of the same video, with a leading hyphen dropped from the video ID somewhere upstream.

## What "Done" Looks Like

Six scripts, each independently scheduled, each maintaining its own seen-list so re-runs don't duplicate content, each failing in isolation rather than taking the others down with it. An enrichment pass turning accumulated notes into an actual graph rather than a flat pile. A Crawl4AI fallback that catches the JS-rendering gap the seen-list used to quietly paper over. And one source — Reddit — parked and honest about not currently working, rather than quietly removed and pretended it was never attempted.

If you're building this yourself: start with one source, get its dedup logic genuinely solid before adding a second, and treat every external API you depend on the way Reddit taught me to — as something that can change its terms out from under you with no warning, not a fixed foundation you get to assume stays put.

Chapter 17b covers what this pipeline was actually building toward — a companion website, and how far I'd gotten with it by the time this book stops. Chapter 18 is where several pieces of this chapter went wrong before they went right — including one afternoon where two completely unrelated-looking failures turned out to share the exact same root cause.
