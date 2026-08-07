---
title: "Agent Harnesses in 2026: OpenClaw vs. Hermes, and What Actually Talks to Your Local Model"
date: 2026-07-15
draft: false
description: "The layer above your inference server — what actually talks to a local model, and what it takes to trust it."
tags: ["agents", "openclaw", "hermes", "local-ai"]
series: ["Local AI Reference Guides"]
---

The layer above your inference server — what actually talks to a local model, and what it takes to trust it.

<!--more-->

## The Layer Above the Server

The first three guides in this series covered hardware, models, and the inference server that ties them together. This one covers the layer sitting on top of all of it — the piece that turns "a model I can chat with" into "a system that goes and does things." It's the newest, fastest-moving part of this whole stack, and it's also the part most likely to be out of date by the time you read this. I've said that caveat before in this series, but it applies here more than anywhere else — the two projects at the centre of this guide changed places in the market inside a matter of weeks earlier this year, and neither had existed in any recognisable form eighteen months before that.

Worth being honest about the pace here before going any further: this is a genuinely new category. A year ago, "agent harness" wasn't a phrase most people would have recognised. Today there are two projects with a combined multi-hundred-thousand GitHub stars, a security incident significant enough to draw comment from Cisco's security team, and a usage flip dramatic enough to be worth its own headline — all inside roughly the same twelve months. Treat everything in this guide as a snapshot, not a settled verdict.

## What an Agent Harness Actually Is

A chat interface answers a question. An agent harness is given a goal, breaks it into steps, calls tools to execute those steps, checks its own work, and keeps going until the job is actually finished — or it decides it's stuck and asks you. That's a genuinely different piece of software from anything covered in Article 3. Lemonade, Ollama, and llama.cpp serve a model. OpenClaw and Hermes are two of the most prominent tools that *use* a served model to actually get things done: managing a smart home, clearing an inbox, running multi-file coding tasks, coordinating a team of sub-agents on a larger project.

They're not competitors to your inference server — they're consumers of it. Point either one at a local model instead of a cloud API and the harness doesn't know or care that "the cloud" is a box on your own network, which is exactly the API-compatibility principle Article 3 spent so much time on.

The practical distinction worth internalising, because it decides whether you actually need this article at all: if what you want is a smarter conversation, you don't need a harness — a well-configured chat interface talking to a good local model, covered fully in Articles 2 and 3, does that job. Harnesses earn their complexity when the task has multiple steps, needs to touch real tools (files, calendars, smart-home devices, messaging platforms), or needs to run unattended and report back. That's a genuinely different use case, with genuinely different failure modes, and it's worth being clear-eyed about which one you actually have before adding this layer of complexity to a setup that might not need it.

## MCP and A2A: The Protocol Layer Underneath Both

Before getting into OpenClaw and Hermes specifically, it's worth understanding the standard both of them — and virtually everything else in this space — are increasingly built on, because it explains a lot about why agent harnesses look the way they do in 2026.

The Model Context Protocol (MCP), Anthropic's open standard for connecting models to tools and data, has been adopted essentially industry-wide — across Anthropic, OpenAI, Google, and Microsoft's own agent platforms, with adoption figures running into the hundreds of millions of downloads. What it actually does: separates "the model" from "the infrastructure it can act on." Instead of a harness needing custom, hand-written integration code for every tool it might touch, it connects to lightweight MCP servers — a filesystem server, a database server, a browser server — each exposing a standard set of capabilities the model can call directly.

Agent2Agent (A2A), Google's complementary protocol now under Linux Foundation governance with 150+ supporting organizations, addresses a different problem: how separate agents — potentially built by different teams, running different models — discover and coordinate with each other rather than one monolithic system trying to do everything itself.

Here's the framing worth being precise about, because it's easy to overstate: MCP and A2A aren't a third contender that beats OpenClaw and Hermes at their own game. They're the plumbing both of those platforms increasingly run on. A harness that speaks MCP natively doesn't need to hand-roll a Home Assistant integration or a Slack connector — it just needs an MCP server for that tool to exist, and increasingly, one already does. The real shift isn't "frameworks are dying," it's that less of a harness's value now comes from its own bespoke integration code and more comes from how well it plays with an ecosystem of standard tool connections it didn't have to build itself. Worth knowing before your next MCP server install, if you're the kind of reader already running a few through Claude Desktop: you're already part of this shift, whether or not you'd put a name to it.

A concrete example makes this less abstract. Say you want an agent to check your calendar, draft a reply to an email, and post a summary to a Slack channel — three completely different systems. The old approach meant a harness needed dedicated, hand-written code for each of those three integrations, maintained separately, breaking separately when any of the three changed their API. The MCP approach: three lightweight MCP servers, each exposing a small, standard set of tools for its respective system, and the harness — any MCP-compatible harness, not just one specific one — calls all three the same way it'd call any other tool. Swap which harness you're using and the MCP servers don't need to change at all. That portability is precisely the same argument Article 3 made about inference servers, one layer further up the stack.

## OpenClaw in Depth

OpenClaw's origin story is almost a case study in how fast this space moves. Austrian developer Peter Steinberger published it as a side project in November 2025 under the name "Clawdbot." A trademark complaint from Anthropic forced a rename to "Moltbot" in late January 2026, then to "OpenClaw" three days after that. None of the renaming slowed it down — the relaunch under its final name ignited explosively, pulling in 100,000 GitHub stars within 48 hours, and by April it had overtaken React as the most-starred repository in GitHub's history.

Architecturally, OpenClaw thinks in terms of agent organisations rather than a single agent: a gateway-first control plane where individual agents act as workers inside a larger coordinated system, capable of holding state and communicating with each other across sessions. Its defining feature is ClawHub, a community marketplace of "claws" — pre-built skills anyone can install rather than write themselves — home automation integrations, messaging platform connectors, scheduling tools, reportedly running into the thousands. Version 4.0, shipped in February 2026 as what Steinberger called "The Agent OS," was a genuine architecture rewrite: a proper gateway daemon, a canvas system, support for over a dozen messaging platforms, cron-based scheduling. It later went native on iOS and Android.

In mid-February, Steinberger announced he was joining OpenAI, and stewardship passed to a newly formed nonprofit, the OpenClaw Foundation — backed by the University of Michigan as its largest donor, with OpenAI, Nvidia, and Tencent among its partners. Reports suggest Steinberger still calls the technical shots day to day, which is a fairly unusual arrangement for a project that size.

**The part that actually matters most for anyone thinking about self-hosting it:** March 2026 saw nine CVEs disclosed in a single four-day window, including one rated 9.9 out of 10 — an authentication bypass in certain channel configurations — and a separate remote gateway exploitation vulnerability rated 8.8. All were patched, but security researchers subsequently found over 135,000 publicly exposed OpenClaw instances across 82 countries via a routine Shodan scan. A separate incident that January — dubbed "ClawHavoc" by the community — found 341 malicious skills through typo-squatting in an initial audit of roughly 3,000 ClawHub listings, a malware rate of around 12% in that scan. ClawHub now requires VirusTotal scanning on submissions as a direct result. Cisco's security team reportedly described personal AI agents like OpenClaw as "a security nightmare" around the same period.

None of that means the project is unmaintained — quite the opposite. At its peak, OpenClaw shipped 62 tagged releases in a single 30-day window with an 89.9% issue close rate, genuinely responsive maintenance by most open-source standards. But that same velocity has a cost: at that release cadence, roughly a quarter of updates reportedly broke message delivery on at least one supported channel. Fast and thoroughly tested aren't the same thing, and OpenClaw's history through the first half of 2026 is a fairly clean illustration of that tension.

## Hermes Agent in Depth

Hermes comes from Nous Research, the team behind the long-running Hermes model series, founded in 2023. The repository itself dates to July 2025, developed quietly for roughly eight months before its first public release in March 2026, under the MIT license. Where OpenClaw grew a marketplace of human-written skills, Hermes took the opposite bet: a closed learning loop that writes its own.

After completing a task, Hermes analyses the steps it took, identifies which parts were genuinely reusable, and converts successful workflows into skill files that load automatically the next time a similar problem comes up — no community marketplace required, no human writing the integration first. Run the same category of task repeatedly and Hermes gets measurably faster and more reliable at it over time, which is a meaningfully different value proposition from "browse a catalogue and install what someone else wrote." Its most-downloaded self-generated skill is reportedly one that improves its own capability-generation process — a slightly recursive detail that tells you something about where its design priorities sit.

Architecturally, Hermes leans toward one agent that improves, rather than OpenClaw's model of many agents that coordinate: a parent process can spin up isolated sub-agents for parallel execution, but — in a genuine fork from OpenClaw's design — those sub-agents don't talk to each other directly. A SOUL.md file defines the agent's identity and communication style globally across every project rather than per-workspace, so its personality stays consistent wherever you use it. Its skill ecosystem, hosted at the open standard agentskills.io rather than a proprietary marketplace, is smaller than ClawHub by a wide margin but has a stricter submission process and a noticeably higher average quality bar as a result.

Release velocity has been genuinely rapid: six numbered releases in the first fifty days, with contributor and merged-PR counts climbing sharply release over release. The May 2026 "Tenacity" release alone shipped 588 merged PRs from 295 contributors, added a durable multi-agent task board, patched eight priority-zero security issues proactively, and added Google Chat as its 20th supported messaging integration. A later release refactored a single 16,000-line core file down into fourteen focused modules — the kind of unglamorous maintenance work that suggests a team thinking past the initial growth spurt. June's "Surface" release brought a native desktop app across macOS, Linux, and Windows.

On security specifically: no agent-specific CVEs have been publicly reported against Hermes through the first half of 2026, across multiple independent sources. Worth stating the honest caveat plainly rather than turning that into a selling point: that reflects real newness and smaller attack-surface exposure so far, not necessarily proven hardening under the same scale of scrutiny OpenClaw has now been through. Container-level security is genuinely solid on paper — read-only root filesystem, dropped capabilities, per-skill permission scoping, credentials handled through environment variables rather than stored plaintext — but a track record under real adversarial pressure is exactly the thing OpenClaw now has, for better and worse, that Hermes doesn't yet.

## Architecture Comparison

| | OpenClaw | Hermes Agent |
|---|---|---|
| Core model | Multiple coordinating agents (organisation) | One agent that improves over time |
| Tool acquisition | Community marketplace (ClawHub) | Self-generated skill files from completed tasks |
| Skill ecosystem size | Thousands of listings, variable quality | Smaller (agentskills.io), stricter submission bar |
| Language/stack | TypeScript/Node.js, Electron-backed | Python |
| Sub-agent communication | Agents can talk to each other directly | Sub-agents run isolated, no direct cross-talk |
| Governance | Nonprofit foundation (post-Feb 2026) | Nous Research, commercial-adjacent |
| Security track record | Serious CVE cluster (patched), high exposure found | No reported CVEs yet — limited data, not proof |
| Messaging platforms | 15-24+ depending on version | 20+ and growing fast |
| Native desktop/mobile | Yes, both | Desktop app added June 2026 |

## Pointing Either One at a Model You Host Yourself

This is where the whole series actually connects. Article 3 covered why API compatibility matters more than any single benchmark, and the same principle applies one layer up: both of these harnesses can be pointed at a local inference server instead of a cloud API, provided the server speaks the right shape of API.

Worth being precise here, because the exact mechanism differs by which Claude surface you're using, if that's part of your stack — and it's easy to get this wrong. Claude Code, the terminal tool, connects to a local server directly: set `ANTHROPIC_BASE_URL` to point at Lemonade, Ollama, or LM Studio, and every request goes there instead of Anthropic's cloud, no proxy required once the server speaks the Anthropic Messages API natively — which, as covered in Article 3, is exactly why Lemonade shipping that endpoint matters. Claude Cowork has the equivalent capability through what Anthropic documents as "Cowork on 3P" (third-party inference) — officially supported, though still a research preview, with one real limitation: Anthropic's own Connectors go unavailable in that mode, since they depend on Anthropic's infrastructure specifically, with MCP servers as the documented replacement. Plain chat — claude.ai, or standard Desktop chat outside the Code and Cowork tabs — has no backend swap available, by design.

OpenClaw and Hermes both work on the same underlying principle as Claude Code: they expect an API in a particular shape, and if your local server speaks that shape, they don't know or care that inference is happening on your own hardware rather than someone else's data centre. The practical upshot for this whole four-article series: hardware (Article 1), model (Article 2), and inference server (Article 3) all feed into this final layer without you needing to touch a cloud API key anywhere in the stack, if that's the setup you want.

## Security Considerations Before You Self-Host Either One

I've spent two decades in enterprise IT and cybersecurity sales, and OpenClaw's spring gave me a genuinely useful, if slightly uncomfortable, case study to point at: a fast-growing platform outrunning its own security defaults is one of the most common patterns I've watched play out with far more expensive infrastructure than a home lab, and it played out here in miniature, in public, with a CVE trail to prove it.

The lesson isn't "pick Hermes because it hasn't been breached yet" — that's mistaking absence of evidence for evidence of absence, and a platform with a smaller install base and shorter track record simply hasn't had the same adversarial attention pointed at it. The actual lesson, consistent with the "prove it yourself, don't trust the spec sheet" thread running through this whole series: audit whichever harness you choose before exposing it to your network, regardless of which one you pick. Check what ships open by default. Check what a skill or ClawHub entry can actually access before you install it — don't assume marketplace presence means vetted. If you're running either of these anywhere reachable beyond your own LAN, put real thought into what "reachable" actually means before 135,000 other installations quietly demonstrate it for you.

## Decision Matrix by Use Case

- **You want the widest possible range of ready-made integrations, and you're comfortable auditing what you install:** OpenClaw. ClawHub's breadth is real, even with the quality caveats.
- **You want an agent that gets measurably better at your specific repeated workflows without you curating a plugin list:** Hermes. The self-improving loop is a genuinely different value proposition, not just marketing language.
- **You're running this on hardware and a network you'd rather not expose to a repeat of March 2026:** lean Hermes for now, but keep auditing regardless — track records shift, and this one's short.
- **You want multiple coordinating agents that can talk to each other on a larger project:** OpenClaw's architecture is built for this directly; Hermes's isolated sub-agents aren't.
- **You don't want to choose at all:** a growing pattern worth knowing about is running OpenClaw as the orchestrator — planning and decomposing larger tasks — with Hermes handling the fast, repeatable execution loops underneath it. Not an either/or for everyone; a genuine hybrid a meaningful slice of the community has already landed on.

One honest note on the popularity question, because it's not as clean as headline usage numbers suggest: OpenRouter data shows Hermes overtaking OpenClaw on daily token volume around May 2026, and the gap widened sharply over the following weeks. But community sentiment analysis of over a thousand Reddit comments found roughly a third of users sticking with OpenClaw despite its rockier year, citing the sheer breadth of its integrations as worth the tradeoff — and a meaningful subset expressed genuine skepticism about how organic Hermes's promotion on Reddit actually was. Usage numbers moved fast. Community trust moved more slowly, and unevenly. Read any single "X has won" claim in this space, including anything printed here, with exactly that in mind — and check the current numbers before you commit to either one, because this is the fastest-moving comparison in the whole series and today's snapshot has a short shelf life.

## Getting Started Practically, Without Over-Committing

Neither of these is a small install-and-forget decision the way pulling a model in Ollama is, so it's worth approaching cautiously rather than diving straight into a full production setup.

Start on a machine you're comfortable being wrong about — a spare box, a VM, a Docker container with limited network access, not the same server holding your Kopia backups or anything else business-critical. Give it a genuinely narrow first task: one messaging platform, one or two tools, nothing touching sensitive data or irreversible actions. Both projects document their default permission models clearly; read that documentation before your first run, not after something unexpected happens. Watch what it actually does for the first few sessions rather than setting it to fully autonomous mode and walking away — the "Auto" approval mode in tools like this genuinely means what it says, and OpenClaw's spring is a reasonable argument for staying in a manual-approval mode longer than feels necessary at first.

Once you trust the basics, connecting it to your actual local inference server — Lemonade, Ollama, whichever you settled on in Article 3 — is usually the natural next step, and it's where the real value of self-hosting shows up: the same agentic capability, without a per-token cloud bill and without your tasks, files, or conversations leaving your own network.

## Mistakes Worth Avoiding

A handful of errors that come up repeatedly with both platforms, worth checking before you commit to either:

- **Installing marketplace skills without checking what they can actually access.** ClawHub's post-audit verified listing count and its raw submission count are two very different numbers, and "available in the marketplace" isn't the same as "vetted." Check permissions before installing, every time, not just for anything that looks obviously risky.
- **Setting full autonomous approval mode on day one.** Both platforms support a manual-approval mode specifically so you can see what an agent actually does before trusting it to do it unsupervised. Skipping straight to "Auto" on a fresh install is skipping the one step designed to catch a misconfigured tool before it does something you didn't intend.
- **Exposing an instance to the open internet without understanding what "exposed" means for that specific platform.** The 135,000 publicly reachable OpenClaw instances found via a routine Shodan scan didn't get there through some sophisticated attack — mostly through default configurations nobody checked before connecting them to a network wider than intended.
- **Treating a usage-leaderboard snapshot as a permanent verdict.** The OpenClaw-to-Hermes usage flip happened over a matter of weeks. Whatever's ahead by the time you're reading this may not be what's ahead by the time you'd actually finish evaluating it — check current numbers rather than trusting a single article, this one included.
- **Assuming "no reported CVEs" means "secure."** Covered above, worth repeating as its own mistake: a shorter track record under real adversarial attention is not the same claim as a hardened one.

## Where This Leaves the Series

Hardware decides what's possible. The model decides what's good. The inference server decides how easily everything else can reach it. And the agent harness — OpenClaw, Hermes, or whatever's displaced them by the time you're reading this — decides how much of that capability actually turns into finished work rather than a very capable chat window.

Look back across the four pieces and there's a single thread running through all of them, worth naming explicitly now that the arc is complete: at every layer, the thing that mattered more than the headline benchmark was portability — whether a decision made today still holds up once the hardware changes, the model turns over, or the tool you're relying on gets overtaken by something that didn't exist six months earlier. A used 3090 outlasts whichever GPU currently tops a benchmark chart. A model chosen for the right tier outlasts whichever one currently leads a leaderboard. An inference server chosen for API compatibility outlasts a raw speed test on one specific card. And an agent harness chosen with an honest read of its security posture, not just its star count, outlasts a usage-leaderboard snapshot that's already stale by the time it's published. None of that is really about AI specifically — it's the same instinct that two decades of watching technology get oversold teaches you to apply to any of it.

That's genuinely the full stack now, top to bottom, and it's taken four articles and a fair amount of research to get there properly rather than just repeating whatever's trending this week. If you want to see all four pieces actually assembled — the real hardware, the real model choices, the real server configuration, and how agentic tooling sits on top of it, built and running rather than just described — that's what the book and the rest of this site are for.
