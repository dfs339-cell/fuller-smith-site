---
title: "AI Is Not New. Owning It Is."
date: 2026-07-14
draft: true
description: "A buyer's guide to local AI hardware, from the NPU already in your laptop to a clustered rig of dedicated boxes — and why 2026's memory shortage makes this decision harder than it should be."
tags: ["hardware", "buying-guide", "gpu", "apple-silicon"]
series: ["Local AI Reference Guides"]
---

## A Brief History: AI Is Not New

Everyone talks about AI like it arrived in November 2022. It didn't. Alan Turing was asking whether machines could think in 1950, and by the time most of us were born, AI had already had two full boom-and-bust cycles. IBM's Deep Blue beat a reigning world chess champion in 1997. IBM's Watson beat two Jeopardy champions in 2011, on live television, answering natural-language trivia clues. Neither system ran on anything you'd recognize as a modern GPU, and neither used anything like the deep learning techniques behind today's models.

The real turning point came in 2012, when a neural network called AlexNet crushed the competition at an image recognition contest — not because the idea was new, but because it was the first time someone trained a network like it on GPUs instead of CPUs, and the speedup made the whole approach practical. That one result is arguably why Nvidia is the company it is today: CUDA, the software layer that lets ordinary code run on a graphics card, had existed since 2006 looking for a killer application. Deep learning was it.

Everything since — the 2010s wave of image and speech recognition, the large language models, ChatGPT's arrival in the cultural mainstream — has been a continuation of that same GPU-shaped bet, running at ever larger scale, on ever more hardware, mostly rented from someone else's data centre.

Here's the bit I want to be honest about before going any further, because I've spent two decades watching technology get oversold: I've seen this exact hype cycle before. Remember when VoIP and IP telephony were "the future" at the turn of the century? Everyone promised instant transformation. It took over a decade before customers actually saw measurable benefit. AI right now feels eerily similar — hyped as revolutionary, but the real value only shows up when it's applied thoughtfully, matched to an actual problem, rather than bought because it was the shiny thing on this quarter's roadmap. A tool nobody's trained to use properly is the new ping-pong table in the office: flashy, fun for a week, ultimately meaningless. There's a real difference between *having* AI tools and *having* AI capability, and most people — and most businesses — are sitting somewhere between the two without realising it.

Which is the actual point of this article. AI itself isn't new, and neither is the hype around it. What's genuinely new is that, for the first time, an ordinary person can own a piece of it outright — not rent access through an API, not depend on a subscription staying priced the way it is today, but run a genuinely capable model on hardware sitting on their own desk. That's exactly what I've spent the last year doing, building what I've been calling my Sovereign AI Lab.

It didn't start with a grand plan. A year or so ago I installed Ollama on a home PC mostly out of curiosity — it was slow by any modern standard, but it solved something that actually mattered to me: control. Being able to work with sensitive documents and ask real questions without any of it leaving my machine changed how comfortable I felt using AI for anything client-facing. From there it kept escalating — an NPU-equipped machine changed the experience again, and eventually that curiosity turned into building something properly capable rather than just functional. The Sovereign AI Lab is where that journey landed.

A bit of context on why I think I'm worth listening to on this, beyond just having built the thing: I started out as a copier engineer, three years in the field with my hands actually inside the machines, before moving into training rooms teaching print theory and colour copier courses. From there it was two decades in enterprise tech sales — servers, telecoms, now IT and cybersecurity — which means I've sat through more vendor pitches than I care to count, and I can smell a spec sheet that's been dressed up to hide a weak spot from three slides away. That combination is exactly what this guide is: someone who's actually built the thing, filtered through someone who's spent a career telling the difference between genuine capability and a good sales deck.

The hardware decision is the one people get wrong most often — usually by either overspending on capability they'll never use, or underspending and hitting a wall six months in. The rest of this guide is about how to buy into local AI ownership properly, in a hardware market that's currently more volatile — and stranger — than it's been in years.

## AI Already in Your PC (and You Probably Didn't Notice)

Before you spend anything, it's worth knowing that ambient AI has already quietly landed in ordinary consumer hardware — most people just haven't clocked it.

Open Word and look at the Review tab. That Editor pane doing more than spellcheck has been AI-powered for years. Every "Coaching" tip Outlook offers on a draft email is the same underlying system. Teams is arguably the most AI-heavy app most people use daily: background blur has been running a local computer vision model in real time for years, and background replacement is more demanding still — tracking your outline frame by frame and compositing a new scene behind you. On a modern PC with a capable GPU or NPU, that's seamless. On an older machine, you'll see the frame rate drop and the edges go wobbly. That's not a bug. That's your hardware telling you it's struggling with a genuine AI workload.

Nvidia Broadcast deserves special mention — it's one of the most immediately useful pieces of local AI most people with an RTX card have never installed. It's a free app that sits as a virtual camera and microphone between your hardware and any calling app, applying AI processing before the signal reaches the call: background noise removal that's noticeably better than built-in suppression, room echo cancellation, background blur and replacement, and eye contact correction that adjusts your gaze to look at the camera even while you're reading notes off-screen. The catch: it needs a genuine RTX card, not just any Nvidia GPU. If you're on a Copilot+ laptop instead, Windows Studio Effects does the same job through the dedicated NPU, and you may not need Broadcast at all.

This matters because it sets real expectations before you spend a penny: small, specific AI tasks are already solved, cheaply, on hardware plenty of people already own. What this guide is really about is the much bigger jump — running a general-purpose language model, capable of real reasoning and conversation, entirely on your own machine.

## Entry Tier: What Your Existing PC Might Already Do

Three tiers matter here, and it's worth knowing which one you're actually in.

Any decent modern PC handles cloud AI without trouble — Microsoft 365 Copilot, Outlook's draft assistant, Edge's summarise feature, GitHub Copilot. An i5 or Ryzen 5 with 8GB of RAM and a reasonable connection is enough, though 8GB is increasingly the bottleneck it wasn't five years ago: open a browser with a dozen tabs, Teams, Outlook, and Word on a modern Windows machine and you're at 7-8GB before you've done anything else.

NPU-equipped machines — what Microsoft calls Copilot+ PCs — are the newer middle tier. The NPU handles Teams Studio Effects, live captions, and similar always-on background AI without dragging down your main CPU or draining the battery. The current threshold is 40 TOPS; Intel Core Ultra and Qualcomm Snapdragon X chips meet it. 16GB of RAM is the practical minimum at this tier.

A discrete RTX GPU unlocks the interesting territory: Nvidia Broadcast, local language models via Ollama or LM Studio, fast local transcription, image generation. The constraint is VRAM — a 12GB card like an RTX 3060 comfortably runs 7-13B models, genuinely capable for everyday tasks. If a model doesn't fit, it spills into system RAM and slows down noticeably — still functional, just not what you'd want for real-time work.

If you're in that third tier already, you can likely run small quantized models (3-8B) at usable speed through Ollama or LM Studio right now. Not a frontier-model replacement, but genuinely useful for drafting, summarizing, and simple coding help, at zero additional hardware cost. Worth trying before you spend a penny further.

## Why VRAM Matters More Than Clock Speed

Here's the moment most people hit, sooner or later: you're happily running a small model, everything feels fine, and then you try something a bit bigger — a heavier reasoning model, a longer document, a more ambitious project — and it just stops. Not slows down. Stops. CPU and RAM upgrades that solved every other performance problem you've ever had suddenly do nothing, because this isn't that kind of bottleneck.

Once you go beyond small on-device models, the single most important spec is memory — specifically, how much fast memory the model's weights can sit in during inference. A faster chip with too little memory doesn't run the model slowly. It fails to load it at all. This is why the rest of this guide is organized around memory capacity first, raw compute second — and why AI workloads have quietly forced a rethink of what "compute" even means for a lot of buyers. It's no longer just clock speed and core count. It's CPU, GPU, and NPU working together, and increasingly, how much unified or dedicated memory sits behind all three.

## The Elephant in the Room: 2026's Memory Shortage

Here's the thing that makes buying hardware right now genuinely harder than it was even a year ago: 2026 has an active, structural memory shortage, and it's reshaping every category in this guide simultaneously. AI data centres are absorbing the overwhelming majority of global memory production — analysts at IDC estimate data centres could consume as much as 70% of the world's memory output this year, up from roughly a quarter just a few years ago. Samsung, SK Hynix, and Micron, who make nearly all of it, have reallocated capacity toward the high-bandwidth memory used in data-centre AI chips, leaving less for everything else: graphics cards, games consoles, laptops, and Mac Studios all draw from the same shrinking pool.

The knock-on effects are visible everywhere in this guide. The RTX 5090 launched at $1,999 and was trading at $4,300–5,000+ by July. Nvidia's DGX Spark went from $3,999 to $4,699 in a single price increase in February 2026. Nvidia has also skipped a new gaming GPU architecture entirely in 2026 — the first time in roughly three decades it's done so — and shelved a planned RTX 5080 Super refresh because the memory it needed was worth more sold into AI accelerators. This isn't the 2021 GPU shortage, which was a demand-side shock driven by crypto mining and eased within about a year once mining profitability collapsed. This one is a supply-side reallocation backed by the capital budgets of the largest technology companies on Earth, and most analysts don't expect meaningful relief before 2027 or 2028 — new fab capacity takes years to plan and bring online, and even aggressive investment today won't materially shift supply until late 2027.

I've been saying this to business customers since the start of the year, and it applies just as much to a home buyer as it does to enterprise procurement: this isn't a short-term blip you wait out. The assumption that prices "normalise next quarter" isn't valid this cycle. Hardware buying has genuinely become a strategic decision rather than a transactional one — delaying a purchase now carries real cost, because the shortage isn't the kind that reverses itself once a speculative bubble pops. There's no equivalent release valve this time.

## Used Market Considerations, and the Mining Connection

The used GPU market in 2026 is a genuinely strange mix of two opposite crypto stories layered on top of each other, and it's worth understanding both.

**The old story, and why it's good news for you:** Ethereum's move away from GPU-based mining back in 2022 flooded the used market with ex-mining cards, and that supply has never really gone away — prices for a used RTX 3090 have stabilized in the roughly $600–800 range through 2026, holding steady even as new-card prices have gone haywire. Contrary to intuition, a used mining card is often in *better* condition than a used gaming card: miners typically ran their cards at a steady, moderate power draw and constant temperature 24/7, rather than the thermal spikes and cool-downs of gaming sessions. The wear items to check are fans and thermal paste — both cheap to replace. Watch for cards with a modified BIOS (visible in GPU-Z), which isn't dangerous but signals a card that was pushed beyond stock specs.

**The new story, and why it complicates things:** a new proof-of-work cryptocurrency called Pearl launched its mainnet in April 2026, using a mining algorithm built on the same matrix-multiplication math as AI inference — meaning, unusually, that AI GPUs and mining GPUs are now competing for the *same* hardware again, in the opposite direction from 2022. Early per-card mining revenue on an RTX 5090 was reported as high as $33.80/day before network difficulty climbed and roughly halved that. It's a volatile, thinly-traded coin, and a researcher recently published evidence that a meaningful share of "AI mining" on the network may just be miners feeding it random noise to collect rewards without doing any real computation at all — so treat it as a speculative sideshow rather than a durable source of GPU demand, but one worth knowing about if you're wondering why demand for high-VRAM cards hasn't eased even with new-card prices this high.

**The bottom line:** a used RTX 3090 remains the best-value entry point in this entire guide, precisely because its pricing is set by a mature, stable secondary market rather than the new-card supply crunch. It's the sweet spot this guide keeps returning to.

## Single vs. Dual RTX

- **RTX 3090 (24GB), single card** — $600–800 used. Comfortably fits 30B-class models at reasonable quantization. The value anchor of this whole guide.
- **RTX 4090 (24GB), single card** — no longer in production, so all available stock is second-hand; realistically $1,400–1,800. Meaningfully faster than a 3090 at the same VRAM ceiling.
- **RTX 5090 (32GB), single card** — officially $1,999, realistically $4,300–5,000+ as of July 2026. The only single consumer card that comfortably handles very long context windows on larger models, thanks to the extra 8GB over the 4090/3090.
- **Dual RTX 3090s (48GB combined)** — roughly $1,200–1,600 in GPUs alone, before the motherboard, PSU, and PCIe risers a dual-card build needs. This is where you start being able to run 70B-class models at real quantization, something no single 24GB card manages cleanly. Expect a total power draw north of 700W under load, and real thermal/airflow planning if the machine runs continuously in a room you also occupy.
- **Beyond two cards** — the DIY multi-GPU route is the raw-throughput king for anything that fits in the combined VRAM: three used 3090s (72GB combined) will out-token a DGX Spark or a Strix Halo cluster on models under that ceiling, but you're now drawing over 1,000W and building something closer to a small server than a PC.

## Edge Devices: NVIDIA Jetson Orin

The Jetson Orin Nano Super Developer Kit is the outlier in this whole guide on price: $249, with 8GB of memory and 67 TOPS of AI performance. It's not going to run a 70B model, but for robotics, vision, and small on-device AI projects, it's a genuinely remarkable amount of capability for the price, and it's aimed squarely at hobbyists and makers rather than being a scaled-down version of something more expensive.

## Unified Memory APUs: AMD Strix Halo

This is what I actually run, so I'll give it to you straight rather than the usual neutral buyer's-guide hedging.

AMD's Ryzen AI Max+ 395 (codenamed Strix Halo) packs up to 128GB of unified LPDDR5X memory shared between a 16-core Zen 5 CPU and a 40-compute-unit integrated GPU, all inside a mini PC drawing 55–120W. I run Lemonade on mine to serve models locally, and the experience has told me something the spec sheet won't: this isn't a toy. It's genuinely capable of being someone's daily-driver inference server, not a weekend experiment that gets mothballed.

Mini PCs built around it — Framework Desktop, GMKtec EVO-X2, Beelink GTR9 Pro, AMD's own first-party Ryzen AI Halo dev kit — have launched at prices from roughly $1,499 up to $3,999 depending on vendor and configuration, and like everything else with LPDDR5X soldered to the board, pricing's been climbing through 2026 as the same memory squeeze hitting GPUs and Macs reaches this category too. Expect current listings to run noticeably above early-2026 launch pricing, and check live prices rather than trusting anything printed here for long.

Here's the tradeoff I'd want you to actually understand, not just skim past: its 256GB/s memory bandwidth is well behind a discrete GPU's, so it's bandwidth-constrained rather than capacity-constrained. That means it can *load* models no consumer GPU can touch, and MoE-architecture models in particular run at genuinely usable speed — community benchmarks put 120B-class MoE models at 50+ tokens/second — because they only activate a fraction of their total weights per token. Dense 70B models are a different story, dropping to around 4–6 tokens/second, and I wouldn't recommend Strix Halo to someone whose whole workload is dense 70B-class models expecting GPU-tower speed. Know what you're buying it for. If your workload favours MoE-style models — and increasingly, the good open-weight models do — this thing punches well above its price. If it doesn't, a discrete GPU will feel faster for anything that fits in its VRAM, and you should just buy the GPU.

## Dedicated LLM Boxes: NVIDIA DGX Spark

DGX Spark is NVIDIA's purpose-built answer to "I want a box, not a PC I have to configure." It pairs a Grace Blackwell superchip with 128GB of unified memory, delivers up to 1 petaFLOP of AI performance at FP4 precision, and ships with the full NVIDIA AI software stack — CUDA, Ollama, Docker with GPU passthrough — preinstalled. It launched at $3,999 and, caught in the same memory shortage as everything else here, rose to $4,699 by February 2026 — roughly $37/GB of unified memory, notably more than AMD's competing Strix Halo box at the same $3,999 for the same 128GB.

I've spent twenty years selling enterprise hardware, and this is exactly the kind of premium I've watched customers pay before without asking hard enough what it's actually buying them. To be clear: it's not a bad premium. What you're paying for is the CUDA ecosystem and NVIDIA's more mature software stack, and if you're already CUDA-dependent that's a genuinely real line item, not marketing fluff. But if you're not already locked into CUDA tooling, don't let the NVIDIA badge alone talk you out of the box that costs the same and gives you the same memory. Ask what the premium is actually buying you before you pay it — the same question I'd ask any vendor pitching me a "premium tier" in a procurement conversation.

**Who actually needs this over a used 3090:** someone who needs 70B–200B parameter models with full CUDA compatibility, in a box smaller than a textbook, without building or configuring anything. Someone whose workload fits in 24GB, or who's comfortable with Linux and ROCm, will get better value elsewhere in this guide.

## Clustering: When One Box Isn't Enough

This is genuinely one of the more exciting developments in local AI hardware in 2026, and it barely existed as a mainstream option a year ago: you can link multiple unified-memory boxes together over a network and have them behave, from the model's point of view, as one much larger accelerator.

**DGX Spark clustering** is the official, supported route. Two units connect directly via their ConnectX-7 networking at 200 Gbps, creating a combined 256GB memory pool — no switch required for a two-unit setup — enough to run models up to roughly 405 billion parameters.

**Strix Halo clustering** is community-driven but remarkably mature. Using llama.cpp's RPC backend, one machine acts as the controller (handling tokenization and scheduling) while additional machines run lightweight RPC servers that expose their memory and compute over the network — via Thunderbolt/USB4 for a direct two-or-three-machine link, or 10GbE for a larger setup. AMD's own developer team has published an official playbook demonstrating a **four-node Framework Desktop cluster running Kimi K2.5, a full trillion-parameter open-weight model**, entirely on consumer hardware — 512GB of combined memory, built from four mini PCs. Enthusiasts have independently built two-node clusters combining a Framework Desktop and an HP mini workstation, running MiniMax-M2 and GLM 4.6 at usable (if not fast) token rates.

The honest caveat on any clustering setup: you're trading speed for capacity. A model sharded across a network runs slower than the same model sitting entirely in one machine's memory, and setup is a genuinely technical, mostly Linux-first exercise — not something to attempt as your first local AI project. But for anyone who wants to run a model that simply doesn't fit on any single consumer box at any price, clustering two or more Strix Halo or DGX Spark units is now a real, documented path, not a research curiosity.

**Mac mini clustering — a useful counter-example.** Apple Silicon clustering has been tried too, and the results are a genuinely useful contrast to the AMD story above. One widely-cited hands-on test connected up to five M4 Mac Minis over Thunderbolt using Apple's MLX framework and benchmarked small and large models across different configurations. On a small model (Llama 3.2 1B), a single base M4 alone hit around 70 tokens/second, and a single M4 Pro alone hit 95–100 — but two base M4s linked through a Thunderbolt hub actually *dropped* to 45 tokens/second on the same model, because network overhead outweighed the benefit at that scale. A direct Thunderbolt link between two machines (bypassing the hub) recovered to about 95, roughly matching a single M4 Pro. The full five-machine cluster landed at 67–74 tokens/second — still not meaningfully ahead of just buying one better machine. On larger models the pattern held: a 32B model ran at 8 tokens/second on a base M4 and 12 on an M4 Pro; a 70B model managed only 4–5 tokens/second even on M4 Pro units. The one clear win was power draw — the entire five-machine cluster pulled roughly 200W under full load, against 600W+ for a single high-end discrete GPU.

The practical takeaway lines up with the Strix Halo comparison: clustering genuinely works when the goal is *capacity* — running something that flatly doesn't fit on one machine, the way AMD's trillion-parameter demo does — but it's a poor tool for chasing *speed* on models that already fit on a single unit. If a model fits on one Mac mini or one Strix Halo box, buying the single better machine beats clustering several weaker ones almost every time.

## Apple Silicon: The Mac Studio, and the Quiet Demise of the Mac Mini

For years, the Mac mini was the default answer to "cheapest way into local AI with real memory capacity" — Apple Silicon's unified memory architecture means the CPU, GPU, and Neural Engine all share one pool, so a Mac mini with enough RAM could run 70B-class models at low quantization for a fraction of a GPU tower's price and power draw. That's quietly stopped being true in 2026.

Apple has repeatedly cut memory configurations across the Mac mini and Mac Studio lineup this year, citing global memory supply constraints. The 32GB Mac mini configuration was removed from sale entirely by May 2026. The Mac Studio lost its 512GB option in March, then had the 256GB upgrade price increased, then lost the 128GB and 256GB options too — the current M3 Ultra lineup tops out at 96GB. Lead times stretched dramatically in the process: reports in late April described a 64GB Mac mini ordered fresh taking sixteen to eighteen weeks to ship, and a 256GB Mac Studio taking four to five months. Apple's own leadership has confirmed the underlying cause plainly — CEO Tim Cook said the Mac mini and Mac Studio "may take several months to reach supply demand balance," and an Apple product manager has said the company under-forecast demand from people specifically buying these machines to run AI locally.

**What this means practically:** the Mac mini is still a genuinely fine small local-AI box — the M4 Pro configuration with 48GB remains a sensible buy for 14B–35B class models — but it's no longer the effortless, always-in-stock budget path to *large* local models it was a year ago. The Mac Studio M3 Ultra, at $3,999 starting and capped at 96GB, is now the practical ceiling for anyone wanting Apple's unified memory approach at meaningful scale, and even that ceiling is lower than it was six months ago.

**Next-gen outlook:** M5 Max and M5 Ultra Mac Studio models are expected later in 2026, with faster memory bandwidth, but persistent DRAM shortage conditions make it genuinely uncertain whether high-capacity memory options return at launch, or whether pricing simply rises further. If very large unified memory matters to you, buying into the current lineup rather than waiting is a reasonable read of the situation, not a foregone "wait for the next one."

**Apple Silicon vs. discrete GPU, honestly:** unified memory wins decisively on capacity per watt and is dead silent under load. It loses on raw throughput against a top-end discrete GPU, and on ecosystem — CUDA-dependent tools don't run natively on Apple Silicon.

## Cost Per Gigabyte of Unified/VRAM Memory (Mid-2026 Snapshot)

| Platform | Memory | Approx. price | Rough $/GB |
|---|---|---|---|
| RTX 3090 (used) | 24GB | $600–800 | ~$25–33 |
| RTX 4090 (used) | 24GB | $1,400–1,800 | ~$58–75 |
| RTX 5090 | 32GB | $4,300–5,000+ | ~$134–156 |
| Strix Halo mini PC | 128GB | $1,499–3,999 (rising) | ~$12–31 |
| DGX Spark | 128GB | $4,699 | ~$37 |
| Mac Studio M3 Ultra | 96GB | $3,999–5,499 | ~$42–57 |
| Jetson Orin Nano Super | 8GB | $249 | ~$31 |

*Prices move fast in this market — treat this table as a mid-2026 snapshot, not a permanent reference. [Update this table each quarter — see the maintenance schedule.]*

The standout on pure $/GB right now is still the used RTX 3090 and the entry-tier Strix Halo mini PC — both, for different reasons, comparatively insulated from the worst of the 2026 shortage.

## Power, Noise, and Thermals

A full tower with a discrete GPU under sustained load draws 400–500W (700W+ for a dual-card build) and needs real airflow planning if it's running 24/7 in a home office. Strix Halo mini PCs draw a fraction of that — often under 120W — specifically because there's no separate discrete GPU to feed. A clustered setup multiplies whichever baseline you started from by the number of nodes, which is worth factoring in before you commit to four mini PCs running around the clock.

## Don't Trust the Spec Sheet — Prove It Yourself

I've spent a career watching people get sold on a slide deck and disappointed by the actual product. Twenty years in enterprise sales taught me one thing above almost everything else: seeing a system actually do the thing, on hardware you can touch, changes decisions in a way a spec sheet never does. I still pack a physical demo kit to customer sites for exactly this reason — a screen share tells people about a feature, but watching it work in their own environment, on their own problem, is what actually builds confidence.

The same logic applies here, and almost nobody buying local AI hardware does it. Every number in this guide — tokens per second, VRAM ceilings, price-per-gigabyte — is someone else's benchmark, on someone else's workload, at a point in time that's already aging. Before you spend real money:

- If a retailer or a friend has the box you're considering, ask to actually run your model, your real prompts, before you buy — not the benchmark suite the manufacturer chose.
- Community forums (r/LocalLLaMA, the Strix Halo wiki, vendor Discords) are full of people who've already run exactly your use case and posted real numbers, not marketing ones. Search before you buy, not after.
- If you're choosing between two options at a similar price, the one with a more active, more honest community around it is usually worth more than the one with a marginally better spec sheet — support and shared troubleshooting matters more once something inevitably doesn't work first time.

None of that replaces the research in this guide. It's what you do with the research once you've narrowed it down to two or three real options — the last mile between "this looks right on paper" and "I know this actually does what I need."

## Recommendation Matrix

- **Already own it (NPU/Copilot+ PC):** try Ollama or LM Studio with a small quantized model before spending anything.
- **Entry ($0–500):** a Jetson Orin Nano Super if your interest is edge/robotics; otherwise save toward a used 3090.
- **Budget ($600–1,500):** a used RTX 3090 — still the best $/GB in this entire guide.
- **Mid ($1,500–3,000):** entry-tier Strix Halo mini PC, or a dual RTX 3090 build if raw speed on 70B-class models matters more than power draw.
- **High-end ($3,000+):** DGX Spark if you're CUDA-dependent and want a preconfigured box; Mac Studio M3 Ultra if you want the most unified memory Apple currently sells and can live with 96GB.
- **Beyond a single box:** clustering two or more Strix Halo or DGX Spark units, once you're comfortable with Linux and want to run something that genuinely doesn't fit anywhere else — including, as AMD has now demonstrated, a full trillion-parameter model on consumer hardware.

*[Refresh pricing and specs quarterly — this market is moving unusually fast in 2026. See the Maintenance Schedule tab in the content calendar.]*

## The Bottom Line

Buy for the workload you actually have, not the one you think you might have someday. I've sat across the table from enough customers over-buying "future-proof" hardware to know that's usually just an expensive way of putting off a decision — the market moves fast enough in 2026 that whatever you buy today will look dated in eighteen months regardless. Buy what solves the problem in front of you, run it properly, and upgrade when the workload genuinely demands it, not before.

This is the exact decision I made building my own Sovereign AI Lab, and I'm still glad I made it the way I did. Once you've settled on hardware, the next question is which models actually fit on it — that's the next piece in this series. If you want to go deeper still, the full software side — running your own models, keeping the whole thing private and under your control — is what the rest of this site and the book are for.
