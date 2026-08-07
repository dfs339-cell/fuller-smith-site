---
title: "The Best LLMs to Run on Home Hardware — July 2026 Snapshot"
date: 2026-07-17
draft: false
description: "A tier-by-tier snapshot of which open-weight models are actually worth running on the hardware you have, right now — not a permanent ranking, a dated one."
tags: ["models", "buying-guide", "vram"]
series: ["Local AI Reference Guides"]
---

This is a snapshot, on purpose, not a permanent ranking. I've dated it in the title deliberately, because Article 2 already made the case that a "best models" list is stale within months in this space — and pulling this together confirmed that instinct rather than undermining it. Treat this as "what to actually download this week," not gospel. I'll refresh it on the same quarterly cadence as the hardware and VRAM guides.

## The Method Here

Rather than one benchmark or one source, this pulls from five independent July 2026 leaderboards and buying guides, checking where they actually agree rather than trusting any single ranking. Where sources converge — and several do, strongly, on a couple of specific picks — that's a genuinely useful signal. Where they diverge, I've said so rather than picking a winner arbitrarily.

One rule worth stating upfront, echoed across nearly every source: don't start with the biggest model you can theoretically fit. Start with the best model your hardware runs *comfortably*. A smaller model that responds instantly beats a larger one crawling at one token per second, every time, for actual daily use.

## CPU-Only / No Dedicated GPU

**Phi-4-mini-instruct** (Microsoft, MIT license, 3.8B params) is the standout here — genuinely runs on a CPU alone, no GPU required at all. It won't touch Qwen or DeepSeek on hard reasoning, and it's not supposed to. The point is that it runs on hardware you already own, today, with nothing to buy. Good for prototyping, learning the tooling, or a genuinely low-resource machine that isn't getting a GPU any time soon.

## ~8GB VRAM

**Gemma 4 E2B/E4B** (Google) — the edge-tier variant of the Gemma 4 family, built specifically for constrained hardware. Start here with a low-bit quantized build and keep context windows modest; this tier is about getting something genuinely usable running, not chasing benchmark scores.

## ~16GB VRAM

**Gemma 4 12B** is the specific pick multiple sources converge on for this tier — a sensible middle ground before you're into serious GPU territory.

## 24GB VRAM — The Sweet Spot

This is where the most interesting convergence happened across sources, and it's worth taking seriously if you're deciding what card to buy: **Qwen3.6-27B** is the standout pick for a single 24GB card (RTX 3090/4090-class), described by one tracker as "the clear pick" and by another as scoring close enough to models needing ten times the hardware that it "embarrasses much larger models." The actual numbers: 77.2 on SWE-bench Verified, a 262K context window, dense 27B architecture quantizing cleanly to Q4 on one card.

**Gemma 4 31B** is the multimodal alternative at the same hardware tier — worth the swap specifically if you need image understanding, which Qwen3.6-27B and GLM-5.2 both lack.

Worth noting directly: **Qwen3-Coder-Next** — the model I run myself, aliased to my Sonnet tier — shows up independently across this research as specifically well-suited to self-hosted coding servers at this tier, more efficient for that specific job than the larger generalist Qwen3.6-27B. Genuinely good to see independent confirmation of a pick I made before this research existed, rather than working backwards from what I already run.

## 48GB (Dual Consumer GPU)

Past 24GB, you're into dual-GPU territory — two RTX 4090s giving roughly 48GB combined, covered in Article 1's single-vs-dual-RTX section. **Llama 4 Scout** is the standout reason to be here specifically for its long-context capability: a theoretical 10M-token window, though every source is consistent that self-hosted deployments realistically cap around 128K-256K tokens once you account for actual KV cache memory cost — the same context-length math Article 2 walked through in detail. Good pick specifically for large document work, code repository Q&A, or legal/research analysis where context length matters more than raw speed.

## Beyond a Single Rig — 4x A100 and Up

This is where the frontier open-weight models actually live, and where the "home hardware" framing of this article stops applying cleanly — these need genuine multi-GPU infrastructure, not a single desktop.

**GLM-5.1** (MIT license) is the standout MIT-licensed option at this tier — fully open, no usage restrictions, competitive with much less permissively licensed frontier models.

**Kimi K2.5** leads specifically on SWE-Bench Verified (76.8%) at the 4x A100 level — the strongest coding-specific pick if you've got access to that kind of hardware, whether that's owned or rented by the hour.

## The Frontier — 8x H100 Territory

Two models worth knowing about even though almost nobody reading this will run them at home, because they define the actual ceiling of open-weight capability right now:

**Kimi K2.6** — a 1 trillion-parameter MoE (32B active), native INT4 quantization, the highest-scoring self-hostable open-weight model on at least one major tracker. 80.2 on SWE-bench Verified, 89.6 on LiveCodeBench — genuinely frontier numbers. Needs an 8x H100 node. Not a home-lab model by any stretch, but worth knowing it exists if you're ever pricing out rented infrastructure for a specific hard problem.

**GLM-5.2** — covered in real depth in Article 2 already, so I won't repeat the full breakdown here. Short version: benchmarked as statistically tied with Claude Opus 4.8 on real coding tasks in Databricks' own internal benchmark, but the realistic local floor is 223GB+ combined memory even at aggressive quantization. Doesn't support image understanding, which knocks it out of contention versus GLM-5.1 or Gemma 4 for anyone needing multimodal capability at a similar tier.

## What This Table Doesn't Tell You

Every source pulling these numbers flags the same caveat, worth repeating here rather than glossing over: benchmark scores depend heavily on serving configuration, coding agent/harness choice, and token budget — the same "harness matters as much as the model" point Article 2 made from the Databricks research. A model that tops a leaderboard using a custom agent harness may perform differently through Ollama with default settings. Treat every number on this page as directional, not a guarantee you'll replicate exactly on your own setup.

And self-hosting has real operational weight the leaderboards don't score: you own the updates, the security patches, the driver compatibility, the monitoring, the backups. That's not a reason to avoid it — it's the entire premise of this site — but it's worth remembering that "best model" and "best model for you, given what you're actually willing to maintain" aren't always the same answer.

## The Bottom Line

If you've got a single 24GB card, download Qwen3.6-27B this week and see how it handles your actual workload — the convergence across sources on this specific pick is strong enough to trust. If you're specifically doing coding work on that same tier, Qwen3-Coder-Next is the more efficient specialist choice. Below that tier, match the model to what your hardware can run comfortably rather than chasing a bigger download. Above it, you're renting or building serious infrastructure, and GLM-5.1 or Kimi K2.5 are the names worth knowing.

This page gets refreshed quarterly, same cadence as the hardware and VRAM guides — check the date at the top before trusting a specific number months from now.
