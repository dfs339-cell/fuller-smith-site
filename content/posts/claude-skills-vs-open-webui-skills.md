---
title: "Claude Skills vs. Open WebUI Skills: What 'Skills' Actually Means Across the Stack"
date: 2026-07-18
draft: true
description: "Three different products call this 'Skills.' They're not the same thing — and knowing the difference matters if you're running Claude Code alongside Open WebUI, the way this whole site does."
tags: ["skills", "claude", "open-webui", "local-ai"]
series: ["Local AI Reference Guides"]
---

"Skills" has quietly become one of those words that means genuinely different things depending on which product you're standing in front of. Claude has Skills. Open WebUI has Skills. OpenClaw and Hermes, covered in the last article, have their own skill marketplaces too. Same word, three meaningfully different mechanisms — and if you're running Claude Code on your personal track alongside Open WebUI on your friends track, the way this whole site has been built, you're actually using two of them at once without necessarily realising they're not the same thing under the hood.

<!--more-->

## Claude Skills: Filesystem-Based, Progressive Disclosure

Claude's Skills are reusable, folder-based resources — a `SKILL.md` file plus optional supporting scripts and reference material, sitting in a directory Claude can read from. Anthropic ships pre-built Skills for common document work (PowerPoint, Excel, Word, PDF creation), and you can write your own for anything specific to how you work.

The mechanism worth understanding is what Anthropic calls progressive disclosure: Claude doesn't load every Skill's full content into context all the time. It reviews what's available, decides what's relevant to the task in front of it, and only loads the detail it actually needs — which is exactly the design choice that keeps a large Skills library from silently eating your context window the way a pile of always-on system prompts would.

A genuinely useful detail: the same Skill runs across Claude.ai, Claude Code, and the API without modification. Write it once, and it works everywhere you use Claude — including, relevantly for this site, inside Claude Code pointed at your own local model via the aliasing setup covered in the book.

Claude Skills can also execute actual code, not just supply instructions — the built-in document Skills work this way, generating real PowerPoint or Excel files rather than just describing what one should contain. That's a meaningfully more capable mechanism than a plain-text instruction set, at the cost of needing code execution enabled and a bit more trust in what a Skill actually does when it runs.

Worth flagging plainly, since it's exactly the kind of thing this site cares about: only use Skills from sources you trust — yourself, or a source you'd trust with genuine code execution on your behalf. A malicious Skill isn't a hypothetical risk category; it's the same trust boundary as installing any other software that runs with your permissions.

## Open WebUI Skills: Markdown-Only, No Code Execution

Open WebUI's Skills system looks similar on the surface — reusable, named, invoked on demand — but it's built on a deliberately narrower mechanism. Open WebUI Skills are plain markdown instruction sets: code review guidelines, a writing style guide, a troubleshooting playbook, a data-analysis workflow. No Python, no API calls, no deployment step. If you can write a document, you can write a Skill.

The lazy-loading mechanism is conceptually similar to Claude's progressive disclosure but implemented differently: when a Skill is bound to a model, only a lightweight manifest (name and description) gets injected into context by default. The model loads the Skill's full instructions on demand, through a dedicated `view_skill` tool, only when it actually decides the Skill is relevant — same underlying goal as Claude's approach (don't blow the context window on Skills you're not using this turn), different plumbing.

In actual use, there are two ways to bring a Skill into a conversation: type `$` in the chat input to open a Skill picker and inject one on the fly, or bind a Skill permanently to a specific model so it's always available without you needing to remember to invoke it — the natural choice for something like "always follow these code review guidelines" on a coding-focused model.

**The deliberate limitation, and why it's a feature, not a gap:** Open WebUI draws a hard line between Skills (markdown instructions, no code execution) and Tools/Functions (which do execute real Python on your server). That's a meaningfully different trust model than Claude's code-executing Skills. Open WebUI's own documentation is blunt about the actual risk here: granting someone the ability to create or import Tools is "equivalent to giving them shell access to the server." Keeping Skills instruction-only, and pushing anything that needs real execution into the separately-gated Tools/Functions system, is a sensible way to let more people write and share Skills without each one being a potential server compromise — directly relevant if you're running the friends track for people you trust with a chat interface but not with server access.

## Where the Two Systems Actually Meet

There's a live, ongoing conversation in Open WebUI's own community about closing the gap between these two mechanisms — a GitHub discussion proposes letting Open WebUI directly mount and read the same filesystem-based `SKILL.md` folders Claude and OpenAI already use, rather than maintaining an entirely separate format. The reasoning laid out there is worth knowing about if you're tracking where this is heading: office/document Skills (spreadsheets, PDFs, DOCX) are the obvious first case, since duplicating Anthropic's own document-handling Skills inside Open WebUI's format would just be redundant work solving an already-solved problem.

Nothing's shipped from that discussion as I'm writing this, and I wouldn't bet on the exact shape it takes — but the direction is telling. The "Skills as a folder with Markdown and resources" pattern is becoming enough of a shared standard that even conceptually different products are looking at ways to speak the same file format rather than reinventing it.

## The Wider Marketplace: ClawHub, agentskills.io, and Now LobeHub

Article 4 already covered ClawHub (OpenClaw's skill marketplace, thousands of listings, real security growing pains) and agentskills.io (Hermes's smaller, stricter-vetted skill standard) in depth, so I won't repeat that ground here. Worth adding to that picture, though: a third marketplace has emerged since — **LobeHub Skills Marketplace** — and it's specifically cross-pollinating between ecosystems rather than staying siloed to one harness. One listing worth noting directly: a skill package that lets an OpenClaw instance control Open WebUI directly — chat completions, model listing, RAG file uploads, knowledge base management, even proxying Ollama's generate/embed/pull commands through Open WebUI's own API.

That's a genuinely interesting shape for where this is heading: not just "which harness has the best skill marketplace," but skills that specifically bridge two different tools in someone's stack — an agent harness on one side, a chat interface on the other, talking to each other through a purpose-built integration skill rather than you manually relaying information between them.

## Practical Guidance: What Actually Matters for Your Setup

If you're running the two-track architecture this whole site is built around — Claude Code/Desktop on the personal track, Open WebUI on the friends track — here's the honest, practical takeaway rather than a feature-by-feature scorecard.

**For your own use (personal track):** Claude Skills are worth building out if you find yourself repeating the same instructions across conversations — a particular document format, a coding convention, a personal workflow. The code-execution capability is genuinely more powerful than a plain-text instruction set, and the cross-surface consistency (same Skill on Claude.ai, Claude Code, API) means it's a one-time investment.

**For the friends track:** Open WebUI's markdown-only Skills are the safer default if you're letting other people create or suggest Skills, precisely because they can't execute code — someone writing a bad Skill gives you a badly-instructed model, not a compromised server. If you want actual executable capability on the friends track, that's what Chapter 10's Pipelines and the separately-gated Tools/Functions system are for, deliberately kept apart from what ordinary Skills can touch.

**Don't assume portability.** A Skill written for Claude won't run in Open WebUI and vice versa, at least not yet, despite the surface-level similarity and the community conversation about closing that gap. If you write something useful for one system, budget time to rewrite it for the other rather than assuming a copy-paste job.

## The Pattern Worth Naming

Across Claude, Open WebUI, OpenClaw, and Hermes, the same underlying idea keeps reappearing independently: package reusable expertise as a small, discoverable, on-demand-loaded unit rather than either repeating instructions every conversation or permanently bloating a system prompt with everything you might ever need. Four different teams arrived at meaningfully similar architecture without obviously copying each other — which is usually a decent sign an idea is actually load-bearing rather than just fashionable.

Where they genuinely differ is trust boundary: how much a Skill is allowed to actually *do*, versus how much it's just telling the model *how* to think about a task. That's the question worth asking before you write or install one, regardless of which product you're doing it in.
