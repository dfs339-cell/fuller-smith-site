---
title: "Matching LLMs to Your VRAM: A Practical Sizing Guide"
date: 2026-07-15
draft: false
description: "Stop guessing whether a model will fit — the actual math, real recommendations per VRAM tier, and when self-hosting stops making financial sense at all."
tags: ["models", "vram", "quantization", "buying-guide"]
series: ["Local AI Reference Guides"]
---

## The Download That Doesn't Fit

Everyone who's tried local AI has done this at least once: found a model that sounds perfect on paper, downloaded 40GB of weights over an evening, and watched it fail to load. Not slowly — instantly. Out of memory, no partial credit. Article 1 covered why VRAM matters more than clock speed. This one is the actual arithmetic: how to know what fits before you download a single byte, what quantization actually costs you in quality, and — because the market's moved fast enough that this is now a genuine question — when buying hardware to self-host stops making financial sense at all.

I said in the last guide I'd rather test something than trust a spec sheet. The numbers in this one are exactly that: real quantization data, a real benchmark from a company that measured this properly on their own codebase, and real hardware requirements for a couple of the most talked-about open models of the last month, checked against multiple independent sources rather than repeated from a single blog post.

## How VRAM Requirements Scale With Parameter Count

The starting arithmetic is simple, then it isn't. A model's weights need roughly 2 bytes per parameter at FP16 (the common training/full-precision format), 1 byte at INT8, and about 0.5 bytes at 4-bit quantization. A 7B model is roughly 14GB at FP16, 7GB at INT8, and 4-5GB at 4-bit. Scale that up and a 70B dense model needs around 140GB at FP16 — completely impossible on consumer hardware — but drops to roughly 40GB at 4-bit, which suddenly fits on a single 48GB setup or two 24GB cards.

The complication is that not every model is "dense." Mixture-of-Experts (MoE) architectures — increasingly the norm for the largest open models — only activate a fraction of their total parameters per token, routing each request to specialized sub-networks. This is a compute story, not a memory story: every expert still has to be resident in memory even though only a handful fire on any given token, because the router can only choose from experts that are already loaded. A 30B-A3B model (30B total, 3B active) needs memory sized for the full 30B, but runs at speeds closer to what you'd expect from a 3B model — which is exactly why MoE architectures have quietly taken over as the practical sweet spot for local hardware with generous memory but modest bandwidth, like the unified-memory boxes covered in Article 1.

## Quantization: What You're Actually Trading Away

Quantization is the single most consequential decision in this whole guide, and it's also the most misunderstood. The short version: dropping precision shrinks the model and speeds it up, and the quality loss is far smaller than the percentage drop in size suggests.

**FP16 / BF16** — full precision, 2 bytes per parameter. The baseline everything else is measured against.

**INT8** — 1 byte per parameter, roughly half the size, generally considered close to lossless for most tasks.

**GGUF/INT4 (Q4_K_M and friends)** — around 0.5-0.6 bytes per parameter, the widely recommended default for local inference. This is where most people should start: roughly a 4x size reduction from FP16 with quality loss that's genuinely hard to detect in normal use.

**Below Q4** — this is where it gets interesting, and where a lot of received wisdom is outdated. Modern "dynamic" quantization (Unsloth's approach is the clearest public example) doesn't quantize every layer equally — it identifies which layers matter most for output quality and leaves those at higher precision while compressing the less important ones aggressively. The result, tested on GLM 5.2: a 1-bit dynamic quant retains around 76% top-1 accuracy while being 86% smaller than the full model, and — this is the bit that surprises people — that 76% figure does *not* mean the model is 24% more wrong. It measures word-choice distribution shifts (whether the model says "I will now..." vs "The answer is..." to start a response), not factual accuracy. The actual capability drop at 1-bit is closer to 18-24%, for an 86% size reduction. A 2-bit quant of the same model reaches roughly 82% on the same measure at 84% smaller. Quantization quality has genuinely gotten better than most local-AI folklore accounts for.

The practical takeaway: don't default to Q8 "to be safe." Q4_K_M is the actual sweet spot for nearly everyone, and if a model doesn't fit at Q4, a well-made dynamic 2 or 3-bit quant is very likely a better choice than downgrading to a smaller, weaker model at higher precision.

**On formats, briefly, because the acronyms multiply fast:** GGUF (what llama.cpp, Ollama, and LM Studio all consume) is the correct default for nearly everyone — it supports CPU/GPU hybrid inference, runs on Nvidia, AMD, Apple Silicon, and CPU-only, and needs no calibration data. GPTQ and AWQ are GPU-only formats built for tools like vLLM, offering better throughput if — and only if — the entire model fits in VRAM with nothing offloaded; they simply don't support the CPU-offload safety net GGUF gives you. EXL2 sits in a similar space, favoured for fine-grained per-layer quantization control in ExLlamaV2-based setups. Unless you have a specific reason to chase GPTQ/AWQ's throughput advantage on a model that fits entirely in VRAM, GGUF is the sensible default and the one every tool in this guide's ecosystem already expects.

## Context Length's Impact on VRAM (The Part Everyone Forgets)

Model weights are only half the memory budget. The other half is the KV cache — the running memory of everything in the current conversation — and it scales with context length in a way that catches people out constantly. A model that comfortably fits its weights in 16GB can still run out of memory purely from asking it to hold a 100,000-token conversation, because every token in context consumes cache space on top of the weights sitting there already.

The formula, roughly: KV cache size scales with layers × attention heads × head dimension × sequence length × bytes per element × 2 (for keys and values). You don't need to run that by hand — the practical rule is that doubling your context window can add several gigabytes on top of what the weights alone required, and it gets worse the larger the model. This is exactly why quantizing the KV cache itself (not just the weights) matters: setting `--cache-type-k q4_1 --cache-type-v q4_1` in llama.cpp can stretch usable context roughly 3x at the same memory budget, which is often the difference between a model that handles a real document and one that chokes on it.

Rule of thumb before you buy: check a model's real-world VRAM figure at the context length *you'll actually use*, not the default 4K a benchmark table quietly assumes.

## VRAM Tier Breakdown: What Actually Fits

| VRAM | Realistic ceiling | What lives comfortably here |
|---|---|---|
| 8GB | ~9-13B dense (Q4), 32K context | Small chat models, light coding help, most Copilot+ PC / entry GPU tiers |
| 12GB | ~13-16B dense (Q4) | RTX 3060 territory — genuinely capable everyday assistant |
| 16-20GB | ~14-25B dense (Q4) | Comfortable single-GPU coding and reasoning models |
| 24GB | ~30-35B dense (Q4), or 30B-class MoE with room to spare | The RTX 3090/4090 ceiling — the single-GPU sweet spot |
| 32GB | 35B dense with long context, or larger MoE | RTX 5090 territory |
| 96-128GB unified | 70B dense comfortably, 120B-class MoE at real speed | Strix Halo, DGX Spark, Mac Studio — Article 1's whole unified-memory section |
| 200GB+ | Frontier open-weight MoE at aggressive quant | Clustering, multi-GPU rigs, or renting — see the GLM 5.2 callout below |

Two things worth flagging on this table. First, MoE models punch well above their weight class on speed at a given memory size, for the reasons covered above — a 30B-A3B model at 24GB will feel faster than a 30B dense model at the same memory footprint, even though they need roughly the same space. Second, this table assumes Q4_K_M quantization throughout; if you're running Q8 or FP16, halve or quarter these figures accordingly.

## Multi-GPU and CPU Offloading Basics

When a model doesn't fit on one card, you've got two real options, and they behave very differently.

**Multi-GPU (tensor/expert parallelism)** splits the model's layers or experts across two or more cards, effectively pooling their VRAM. This is what Article 1's dual-3090 build and the clustering section were both doing, just at different scales — the principle is identical whether you're linking two GPUs in one tower or four Strix Halo boxes over a network. The tradeoff is always the same: more total memory, at the cost of some coordination overhead between devices.

**CPU offloading** keeps some layers on the GPU and spills the rest into system RAM, which is dramatically slower to access but effectively unlimited compared to VRAM. This is the mechanism that makes something like GLM 5.2 runnable on a single 24GB GPU at all — not fast, but functional, provided you've got 256GB+ of system RAM behind it. The rule of thumb: hybrid inference is worthwhile once 60%+ of a model's layers fit in VRAM; below that, you're mostly waiting on RAM and the experience degrades fast.

Neither of these is free lunch. Multi-GPU costs you money and power. CPU offloading costs you speed, often dramatically. Which one makes sense depends entirely on whether your bottleneck is budget or patience.

**A concrete example of the offloading math:** take a 70B dense model at Q4_K_M — roughly 40GB of weights. On a single 24GB card, that's 16GB short. llama.cpp's `--n-gpu-layers` parameter lets you offload as many layers as fit and push the rest to system RAM; with a 70-80 layer model, offloading 45-50 layers to a 24GB card and leaving the remainder in RAM is a realistic split. The GPU-resident layers run at full speed; the RAM-resident ones bottleneck on your system memory bandwidth, which is typically an order of magnitude slower than VRAM. The practical result: noticeably slower than a card with enough VRAM to hold the whole thing, but genuinely usable — not the unusable crawl people sometimes expect. The 60%+ rule of thumb from above is exactly this ratio: below it, the RAM-bound portion dominates total time badly enough that the experience degrades past "slower" into "frustrating."

## Model Recommendations Per Tier

Rather than chase "best" model — which changes monthly — here's how to think about the decision, borrowed partly from a genuinely useful piece of research I want to flag properly.

Databricks published an internal benchmark in July 2026 that's more useful than most public leaderboards, precisely because they tested on their own real, multi-million-line production codebase rather than a public benchmark that's likely leaked into training data by now. Their headline finding: models cluster into three rough capability tiers, and for everyday tasks — flipping a config, fixing a small bug — the *medium* tier is genuinely, measurably effective, not just "good enough in a pinch." Their own default had been routing everything to the most expensive model available, and the data told them that was wasteful. The same logic applies locally: not every prompt needs your biggest downloaded model loaded. A smaller, faster model for routine work and a bigger one reserved for genuinely hard problems will usually beat running everything through the heaviest thing that fits.

The other finding worth internalising: token price is a poor proxy for real cost. In Databricks' testing, one model was cheaper per token than a rival but ended up costing *more* per completed task, because it needed nearly twice as many tokens — more back-and-forth, more re-reading context — to reach the same result, and scored worse doing it. Locally, this maps onto a real trap: a smaller quantized model that needs three attempts to get a coding task right isn't actually saving you anything over a larger model that gets it right first time. Speed and size aren't the only variables — how many turns a model needs to finish the job matters just as much.

For coding specifically, the standout data point from that same benchmark: GLM 5.2, an open-weight model, landed in the *top* capability tier, statistically tied with Claude Opus 4.8 on real-world coding task quality, at roughly a third of the API cost. That's a genuinely notable result — an open-weight model, meaning weights you can actually download, benchmarking at frontier quality on real production code rather than a public leaderboard. Which brings us to the catch.

**Putting rough names to tiers, with the caveat that this list ages fast:** for genuinely small/fast models in the 3-8B range — quick drafting, summarising, simple lookups — the current crop of small dense and small MoE models handle these comfortably on almost anything with 8GB+. For coding specifically at a size that fits on a single 24GB card, look for MoE architectures in the 30B-A3B class rather than dense 30B — same memory footprint, meaningfully faster, and the architecture the market has clearly converged on for this exact tier. For general chat and reasoning where you want genuine depth and have 24GB+ to spend, a 30-35B dense model at Q4 is the honest ceiling for a single consumer GPU; beyond that you're into the unified-memory or multi-GPU territory from Article 1. The specific model names in each of these categories turn over roughly every few months as new releases land — which is exactly why this guide leans on the underlying framework (parameter count, architecture, quantization) rather than a "top 10 models" list that's stale by the time you read it.

## A Worked Example: Building a Tiered Setup

Rather than running one model for everything, the more sensible approach — and the one I've actually built around — is tiering: a small, fast model for routine work, and a heavier one reserved for tasks that genuinely need it, both served locally and switched between as the task demands rather than always defaulting to whichever is biggest.

The logic maps directly onto the Databricks finding above. Their business default had been routing everything to the most expensive available model, and the data told them that was wasteful — a large chunk of daily work was routine enough that a mid-tier model handled it just as well, for a fraction of the cost. The same logic applies to a home setup with finite VRAM: rather than running one model that has to be a compromise between "fast enough for quick tasks" and "capable enough for hard ones," running two or three purpose-picked models and switching between them by task gets you both ends properly, provided your hardware has room to hold more than one loaded — or fast enough model-swapping that switching doesn't cost you real time.

Practically, this means picking one small model (a few billion parameters, near-instant responses) for drafting, quick lookups, and anything where speed matters more than depth; one mid-size model (14-35B range depending on your VRAM) as the daily driver for genuine work; and, if your hardware stretches to it, one larger model reserved specifically for the handful of tasks each week that actually need the extra capability. Serve them through whichever inference engine fits your setup — Article 3 in this series covers the actual engine choice — and, if you want tools built against a specific API shape to just work without modification, alias the served models to names those tools already expect. The exact configuration is less important than the principle: match the model to the task, not the task to whatever's already loaded.

## The GLM 5.2 Reality Check

GLM 5.2 deserves its own section rather than a line in the tier table above, because it doesn't fit the same framework as everything else in this guide.

It's a 744-billion-parameter Mixture-of-Experts model (about 40B active per token), MIT-licensed, released June 16, 2026, with a full 1-million-token context window. Production-quality serving needs roughly 744GB of memory — an 8x H200 datacenter node, full stop, not a home rig under any circumstances. But there's a genuinely realistic hobbyist floor beneath that: Unsloth's dynamic quantization gets a usable build down to 239-245GB combined memory (VRAM plus system RAM together, via the CPU offloading mechanism above) at 2-bit, or 223GB at 1-bit while — per the accuracy numbers earlier — still retaining the large majority of the full model's real capability.

Here's the honest ceiling check: a single RTX 3090 or 4090 cannot hold even the 2-bit quantized version in VRAM alone. It can only participate as part of a CPU+GPU offload setup alongside 256GB+ of system RAM, and you should expect single-digit tokens per second — usable for offline, privacy-sensitive one-off tasks, genuinely not a daily-driver coding assistant at that configuration. Real documented home builds sit at two very different price points: a 4x RTX 3090 rig (96GB combined VRAM, plus substantial system RAM for the remainder) is the accessible end; a 4x RTX PRO 6000 Blackwell rig (384GB VRAM, roughly $50,000 all-in) gets you genuine frontier-model performance at home, priced like a car.

Worth connecting back to Article 1 here: the natural home for GLM 5.2's 2-bit quant would be a maxed-out unified-memory Mac Studio — except Apple no longer sells a configuration anywhere near 256GB, thanks to the same 2026 memory shortage covered there. Clustering multiple Strix Halo or DGX Spark units is the other realistic path to this kind of memory outside a rented cloud GPU, and it's the one I'd actually point most readers toward if this model matters enough to chase locally.

**Bottom line on GLM 5.2:** frontier-quality open-weight coding capability that genuinely competes with Opus 4.8, but the realistic entry cost is either serious multi-GPU hardware, a clustered unified-memory setup, or accepting that this particular model is one you rent by the hour rather than one you own.

## When Self-Hosting Stops Making Financial Sense

This is the section I think most local-AI guides skip, and it matters more this month than it did six months ago.

A new model called LongCat-2.0 launched at the end of June 2026 from Meituan — yes, the Chinese delivery-and-services company, not a dedicated AI lab, which is itself a sign of how normalised frontier-model training has become. It's genuinely enormous: 1.6 trillion total parameters (MoE, roughly 48B active per token), MIT-licensed, trained end-to-end on over 50,000 domestic Chinese accelerators rather than Nvidia hardware — the first model at this scale to pull that off. It benchmarks ahead of GPT-5.5 on SWE-bench Pro. It spent two months running anonymously on OpenRouter under the name "Owl Alpha," quietly climbing to the top few models by call volume, before anyone knew who'd built it.

None of that is the actually disruptive part. The disruptive part is the price: $0.75 per million input tokens and $2.95 per million output — undercutting GPT-5.5's $5/$30 and Claude Sonnet 5's introductory $2/$10 by a wide margin, with a flat-rate pack of a billion tokens for around $60 for anyone running it hard through a coding agent.

I haven't been able to find a properly documented local quantization guide for LongCat-2.0 the way Unsloth has published one for GLM 5.2, so treat any VRAM figure for it as an estimate rather than sourced fact. Rough extrapolation from the GLM 5.2 numbers above — LongCat-2.0 is about 2.15x the total parameter count — suggests a realistic local floor somewhere around 450-530GB combined memory even at aggressive quantization. That's beyond a serious home multi-GPU rig and beyond most two-unit clustering setups. Practically, this is a model you rent, not one you buy hardware for, for the overwhelming majority of people reading this.

And that's exactly why it belongs in this guide. Every recommendation elsewhere here assumes self-hosting is worth the hardware outlay — privacy, no per-token bill, no dependency on a subscription's pricing staying where it is. That calculation only holds if the alternative (a metered API) is expensive enough to justify it. When a frontier-competitive model is rentable at $0.75/$2.95 per million tokens with zero hardware cost, the breakeven math genuinely shifts. A used RTX 3090 pays for itself against API costs eventually — but "eventually" depends entirely on your actual usage volume, and it's worth doing that sum honestly before assuming self-hosting is automatically the cheaper path. For low-volume, occasional use, renting access to something like LongCat-2.0 may simply be cheaper than the electricity bill on a 24/7 local rig, let alone the hardware.

This doesn't undermine the case for local ownership — the privacy, control, and no-dependency arguments from Article 1 still stand entirely on their own merits regardless of price. But it's honest to say the API baseline has moved, and pretending otherwise does readers of this guide a disservice.

## Sizing Mistakes I'd Flag Before You Buy or Download

A handful of the errors that actually catch people out, most of which trace back to something already covered above but worth stating plainly:

- **Sizing for the model's default context, not your real usage.** A benchmark quoting VRAM at 4K context tells you almost nothing about whether it'll survive a 32K-token document. Check the number at the context length you'll actually use, every time.
- **Assuming MoE "active parameters" is the memory number.** It isn't. Every expert has to be resident in memory regardless of how few fire per token — size for the total parameter count, not the active count, or you'll be short by a wide margin.
- **Treating Q8 as the "safe" default.** It isn't wrong exactly, but it's usually wasteful — Q4_K_M gets you nearly all the quality at roughly a quarter of the memory, and that headroom is better spent on a bigger model or a longer context window than on precision you likely can't perceive.
- **Buying hardware before checking whether renting is actually cheaper for your usage.** Covered in depth above, worth repeating here because it's the mistake with the biggest financial consequence: work out your real usage volume against current API pricing before assuming ownership is the economical choice.
- **Downloading a multi-GPU-only quant format (GPTQ/AWQ) without checking it fits entirely in VRAM first.** These formats don't support the CPU-offload safety net GGUF gives you — if it doesn't fit, it doesn't run, full stop, with none of the "slower but works" fallback a GGUF quant would give you.

## A Simple Rule of Thumb

For quick mental maths before you go looking anything up: at 4-bit quantization, budget roughly 0.5-0.6GB of memory per billion parameters for the weights alone, then add headroom for context — a rough 10-20% on top for typical use, more if you're running long documents or large codebases through it. A 30B model needs roughly 15-18GB just for weights at Q4; budget 20GB+ once you account for a realistic context window. For MoE models, size the memory for the *total* parameter count, not the active count — the compute is cheap, the memory isn't optional.

This gets you close enough to know whether something's worth downloading. For the exact figure, use a proper calculator before you commit to a multi-hour download.

## Tools to Check Before You Download

A few genuinely useful checks before committing to a multi-hour download:

**Hugging Face model cards** increasingly list VRAM requirements per quantization level directly, particularly for models with an active community around them — check the model card itself before digging through forum posts.

**The GGUF files themselves**, once you've found the right repository, list file sizes that map roughly to memory requirements — file size plus context overhead is your real number. Two quantizers worth knowing by name: bartowski maintains a thorough range from IQ2 through Q8_0 for nearly every popular model with importance-matrix variants; unsloth is known for fast-turnaround dynamic quants with fixed chat templates and the accuracy-preserving approach described earlier in this guide. Both publish within hours of a model's release for anything that gets real attention.

**Community forums, searched before you download, not after.** If a model's being talked about at all, someone on r/LocalLLaMA or the relevant Discord has already run the exact sizing math you're about to do, on real hardware, with real numbers — not the manufacturer's benchmark suite. This is the same "prove it yourself, don't trust the spec sheet" principle from Article 1, just applied to models instead of hardware: a genuine community consensus on "this needs X GB and runs at Y tok/s on a Z card" is worth more than any single blog post's numbers, including the ones in this guide by the time you're reading it months from now.

**A quick sanity check with the rule of thumb above**, before any of that — if the back-of-envelope number already rules something out, there's no need to go further down the research rabbit hole for a model that was never going to fit.

## The Bottom Line

VRAM decides what's possible before speed decides what's pleasant. Get the memory budget right — weights, quantization, and context together, not weights alone — and everything else in this guide becomes a tuning problem rather than a wall you hit at 90% through a download. And increasingly, worth asking honestly before you buy anything at all: does this actually need to live on your hardware, or has the API market moved enough that renting the occasional frontier task makes more sense than owning it outright? Both are legitimate answers. Just make sure you're actually choosing, rather than defaulting to whichever one you'd already decided on before doing the sum.

I sized my own setup using exactly this framework — tiered by task, not by whatever happened to be the biggest download available — and it's held up well as the model landscape has shifted underneath it every few months since. If you want the practical next step — which inference engine actually serves these models well, and why the choice matters as much as the model itself — that's Article 3 in this series. And if you want the full build, not just the theory, that's what the rest of this site and the book are for.
