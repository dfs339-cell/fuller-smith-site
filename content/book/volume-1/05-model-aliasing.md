---
title: "Chapter 5: Model Aliasing"
weight: 5.0
draft: false
---

## Why This Chapter Is the Reason You Bought This Book

Everything up to this point has been solid, necessary groundwork — but honestly, none of it is unique to this book. Installing Docker, setting up Lemonade, pulling a model: any half-decent tutorial covers that. This chapter is different. This is the piece that actually makes Claude Code and Claude Desktop believe they're talking to Anthropic's cloud when they're really talking to a box on your own desk, and it's the single most useful trick in this entire build.

## Why Aliasing Is Necessary

Here's the problem in plain terms. Claude Code and Claude Desktop don't just send a prompt and wait for any response — they send a request that specifies a particular model, by name, because different Claude tiers behave differently and the tools are built expecting that. When you point either tool at a custom API endpoint via `ANTHROPIC_BASE_URL`, the requests it sends still ask for a model named something like a Claude model string. Your local server, quite reasonably, has no idea what that name means, because it's serving a completely different model that's never heard of Anthropic's naming scheme.

You've got two ways to solve this mismatch. You could modify every tool that talks to your server to use different model names — fragile, and you'd be doing it forever, for every new tool you ever add. Or you could make your server answer to the names those tools already expect, once, and never think about it again. That second option is aliasing, and it's the approach this whole book is built around.

## The Duplicate-and-Rename Approach

The mechanism is simpler than it might sound, though it does cost you real disk space — worth knowing upfront, and I'll come back to that tradeoff shortly.

The approach is exactly what it says: take the GGUF file for the model you want to serve under a given tier, make a copy of it, and rename that copy to match the exact model string a tool like Claude Code expects. Lemonade's model scanner sees a file under that name and registers it as an available model — in this case, one that happens to be named like a Claude model.

On Windows, that's a straightforward copy:

```powershell
copy "C:\localllms\qwen3-coder-next-Q4_K_M.gguf" "C:\localllms\claude-3-5-sonnet.gguf"
```

That creates a second file, `claude-3-5-sonnet.gguf`, with an independent copy of the same weights as your original Qwen model file. Once Lemonade rescans its model directory, it registers that file as an available model called `claude-3-5-sonnet` — exactly the name Claude Code is asking for. (My own aliases are named without the dated suffix some Anthropic model strings carry — `claude-3-5-sonnet` rather than `claude-3-5-sonnet-20241022` — which works because Lemonade matched what Claude Code was actually requesting at the time. Check the exact string your own client sends, per the pitfalls section below, rather than assuming either form is universally correct.)

**The real tradeoff, worth being honest about:** unlike a hard link, this genuinely duplicates the file on disk. A 20GB model aliased this way costs you another 20GB, not a clever near-free trick. For most home setups with reasonably generous storage, that's a fair price for the simplicity — a plain file copy has no same-volume restriction, no filesystem quirks to work around, and it's a command anyone can understand and troubleshoot at a glance. If disk space is genuinely tight on your setup, that's worth factoring in before aliasing several large models this way; it adds up faster than it looks like it will on paper.

After copying, restart Lemonade Server (or trigger a model rescan if it supports one without a full restart) so it picks up the new filename. Confirm with the same check from Chapter 4:

```powershell
curl http://localhost:13305/v1/models
```

You should now see the aliased name in the list, alongside the original.

## The Worked Example: My Actual Mapping

Here's the mapping I actually run, and — more usefully than just the names — the reasoning behind which model went to which tier, because the reasoning is what you'll actually reuse even if your specific model choices differ.

**Qwen3-Coder-Next → the Sonnet tier.** Sonnet, in normal Claude usage, is the default workhorse — good at code, fast enough for interactive back-and-forth, the model you reach for most of the day. Qwen3-Coder-Next is specifically a coding-tuned model, strong on exactly the kind of task Sonnet handles for most people day to day, and quick enough on my hardware to feel interactive rather than sluggish. Mapping a coding-specialist model to the tier that handles the most everyday coding work was the obvious pairing, not a compromise.

**DeepSeek-R1-Distill-Qwen-32B → the Opus tier.** Opus is the "bring out the big gun" tier — slower, more expensive in cloud terms, reserved for the problems that genuinely need deeper reasoning. A distilled reasoning model is the natural fit here: noticeably more capable on hard, multi-step problems than the Sonnet-tier model, at the cost of speed, which mirrors exactly how Opus behaves relative to Sonnet in the first place. I reach for this tier deliberately, the same way I'd think twice before burning an Opus-tier cloud query on something Sonnet could have handled.

**Mistral Small 3.2 → the Haiku tier.** Haiku is built for speed — quick, cheap, good for high-volume simple tasks where you don't want to wait. Mistral Small fills that role well: fast responses, lighter resource use, the model I want firing for anything routine rather than tying up the heavier tiers on tasks that don't need them.

The pattern worth taking from this, more than the specific model names: match a local model's actual character to the role each cloud tier plays, not just its rough size class. A model that happens to be "medium-sized" isn't automatically your Sonnet-equivalent if its actual strengths don't match what you reach for Sonnet to do.

## Setting Context Size Per Tier

Aliasing gets a model answering to the right name. It doesn't, on its own, tell Lemonade how much context that tier should actually be allowed to use — and left unset, you're relying on whatever default the model ships with, which may not match what that tier is actually for.

Lemonade lets you set `ctx_size` per model, inside that model's own `recipe_options`. Checked against my own running config, here's what's actually set: the Sonnet tier (Qwen3-Coder-Next) runs at 131072 — 128K — since coding work genuinely benefits from a large context window, holding a real codebase's worth of files in view rather than truncating. The Opus tier (DeepSeek-R1-Distill) runs at 32768 — 32K — enough room for the deep, multi-step reasoning that tier is for, without paying the memory cost of 128K on a model I'm not asking to hold entire files in context the way the coding tier is.

I'll be honest about the third one rather than round it up to a tidier number: checking the Haiku tier's actual config while writing this chapter, it's set to 16384 — 16K — not the 32K I'd had in my head as the intended value. That's worth sitting with for a second rather than quietly fixing before anyone notices, because it's a small, useful example of exactly the kind of drift this book keeps warning about: a setting you're confident is one thing until you actually go and check it, and it's been something slightly different the whole time without causing any obvious problem. Sixteen thousand tokens of context is still perfectly reasonable for the fast, high-volume, simple-task role Haiku is meant to fill — this isn't a broken configuration — but it's not the number I'd have told you it was if you'd asked me before I checked.

The honest takeaway: whatever context sizes you intend to set per tier, verify them against the running config rather than trusting your own memory of what you configured, the same discipline Chapter 14 asks of a backup and Chapter 15 asks of a RAG index. A quick way to check what's actually set, across every registered model at once:

```powershell
Invoke-RestMethod http://localhost:13305/v1/models | ConvertTo-Json -Depth 5
```

That returns every model Lemonade currently knows about, aliased ones included, with each one's actual `ctx_size` sitting inside its `recipe_options` — the ground truth, rather than whatever you remember setting.

## Common Pitfalls

**Filename precision matters completely.** The alias has to match the exact model string a tool is sending, not something close to it. Claude Code and Claude Desktop send specific, sometimes dated model identifiers — `claude-3-5-sonnet-20241022`, for instance, not just `claude-3-5-sonnet`. Get the string even slightly wrong and the tool won't find a match, and you'll get a confusing "model not found" error that looks like a deeper problem than it actually is. Check the exact string the tool is sending — usually visible in its own connection logs or documentation — before creating the copy, rather than guessing at the format.

**Lemonade needs to actually rescan.** A copy created while the server's running won't always be picked up immediately — restart the server, or use its rescan command if one's available, and confirm the new name shows up in `/v1/models` before assuming it's working. I've lost time to exactly this: creating the copy correctly, then testing against a server that simply hadn't refreshed its model list yet, and assuming the copy itself was the problem.

**Copies drift out of sync with the original.** This is the one genuinely different pitfall compared to a hard-link approach, and it's worth taking seriously: if you ever update, re-quantize, or replace the original model file, the aliased copy doesn't automatically follow — it's a separate file, frozen at the moment you copied it. Get into the habit of re-running the copy step any time you update a model that's been aliased, or you'll end up serving a stale version under a name that looks current. I keep a short note of which files are aliases of which originals for exactly this reason, rather than trusting memory six months later.

**Disk space adds up faster than expected.** Covered above, worth repeating as its own pitfall: three aliased tiers at real model sizes is a genuine multiple of your total storage, not a rounding error. Check available space before aliasing a new tier, particularly on a system where the model drive is also doing other work.

**Version strings change.** Anthropic updates its model naming over time, and a tool built against a newer Claude Code release may send a different dated string than the one you aliased against last month. If aliasing that worked stops working after updating Claude Code or Desktop, this is the first thing to check — not a broken Lemonade install, just a model string that's moved on without your alias moving with it.

**Don't alias a model to a tier it can't actually serve.** It's tempting to alias your only decent model to all three tiers just to get everything working quickly. Resist this. Part of the value of proper tiering — covered from Article 2 on the companion site — is that a tool genuinely reaching for "Opus" expects Opus-level depth on a hard problem, and getting Haiku-level output back from what claims to be your heaviest tier is a worse experience than being honest about what you've actually got running.

With this working, your local server now answers to the exact names Claude Code and Claude Desktop expect. Chapter 6 wires up the Gateway configuration that ties this aliasing layer cleanly into the rest of the stack, and Chapter 7 gets Claude Code and Desktop themselves pointed at it.
