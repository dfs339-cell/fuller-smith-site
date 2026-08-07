---
title: "Chapter 6: Wiring the Gateway Config"
weight: 6.0
draft: false
---

## The Two Days I'd Rather You Didn't Lose

I'm going to tell you how this chapter actually went for me, rather than the tidy version, because the detour is more useful than the destination would be on its own.

My first instinct, and probably yours too if you've spent any time around multi-provider AI setups, was to reach for LiteLLM — a well-known proxy that sits between your tools and whatever's actually serving the model, translating requests between different providers' API formats. It's the standard answer to "I want one consistent interface in front of several different backends," and on paper, routing Claude Code through a LiteLLM gateway in front of Lemonade looked like exactly the right, properly-engineered way to do this.

I spent two days on it. Genuinely two full days, going back and forth between LiteLLM's own config and testing against Lemonade, making a change, testing, making another change, testing again — round and round the same handful of settings with nothing actually working. The core of the problem, once I'd finally isolated it: every request arriving at Lemonade needs to carry the exact Anthropic model name — the same name you aliased a GGUF file to in Chapter 5 — so Lemonade can match the request to the right renamed file. LiteLLM's entire reason for existing is translating and remapping model names between providers. That's precisely the behaviour I needed switched off, and I could not get it to reliably leave the model name alone, no matter how I configured it. Every route through LiteLLM either rewrote the name into something Lemonade didn't recognise, or dropped it into a shape that didn't match my aliased filename, and the request would fail or silently fall back to a default model that wasn't the one I'd asked for.

## The Realisation That Ended the Two Days

Here's what finally made me stop: I was trying to solve a translation problem with a translation tool, but the translation problem had already been solved, a layer earlier, back in Chapter 5. The whole point of renaming those GGUF files was to make Lemonade answer to Claude's own model names *without needing anything to translate between two different naming schemes at request time*. Putting LiteLLM in front of that wasn't adding a helpful layer — it was adding a second translation step in front of a problem the first one had already fixed, and the two were actively fighting each other.

Lemonade already speaks the Anthropic API natively — that's been true since Chapter 4. Once a request carries the right model name, Lemonade doesn't need anything translating on its behalf. The gateway I actually needed wasn't a separate tool at all. It was just a direct connection.

I've made this exact mistake before in a different context, with far more expensive infrastructure than a home lab — reaching for the "properly enterprise" solution because it's the recognised right answer in general, without checking whether this specific problem actually needed it. Sometimes it does. Here, it very much didn't, and the two days were the cost of finding that out the hard way instead of questioning the assumption on day one.

## What I Actually Run: Direct, No Middleman

For Claude Code, the connection is exactly as simple as Chapter 4 and 5 already set it up to be — no separate gateway tool sitting in between, just the environment variables pointing straight at Lemonade:

```powershell
setx ANTHROPIC_BASE_URL "http://localhost:13305"
setx ANTHROPIC_API_KEY "local-lemonade-no-key-needed"
```

(Same placeholder-key reasoning as before: Lemonade isn't validating this key against anything, but Claude Code expects the field to be present and non-empty, so it needs *a* value, not the right one.)

Restart any open terminal so the variables actually load, and Claude Code sends requests straight to Lemonade — carrying the real Anthropic model name, matching directly against the aliased file from Chapter 5, no translation layer in the middle to get in the way.

**For Claude Desktop** — specifically the Code and Cowork tabs, since plain Desktop chat doesn't support a custom backend at all — the same direct-connection principle applies, relying on the same renamed-file aliasing to do the work. There's no separate gateway configuration specific to Desktop beyond pointing it at the same local endpoint; the aliasing from Chapter 5 is doing all the real work, the same as it is for Claude Code.

## Verifying the Direct Connection Works

The same verification approach from before still applies, and it's still worth doing before opening any client tool — a plain request shaped exactly like what Claude Code would send:

```powershell
curl http://localhost:13305/v1/messages `
  -H "x-api-key: local-lemonade-no-key-needed" `
  -H "anthropic-version: 2023-06-01" `
  -H "content-type: application/json" `
  -d '{\"model\": \"claude-3-5-sonnet-20241022\", \"max_tokens\": 100, \"messages\": [{\"role\": \"user\", \"content\": \"Say hello in exactly five words.\"}]}'
```

If Chapters 4 through 6 are all correctly wired, this returns a genuine response from your aliased model, under the Sonnet name, with no proxy or translation tool anywhere in the chain. If it doesn't, the same troubleshooting logic from before applies: connection refused points back to Chapter 4, a model-not-found style error points back to Chapter 5's aliasing.

## The Lesson Worth Taking From This

If you're tempted to reach for a proxy or gateway tool because it feels like the properly engineered answer, ask first whether the problem it solves has already been solved somewhere else in your stack. In this build, it had — twice, in fact, once by Lemonade's native API compatibility and once by the aliasing trick itself. Adding a third layer on top didn't make the setup more robust. It just gave the first two something to fight with.

It genuinely comes down to something simpler than two days of config files would suggest: it's asking for a model — give it that model. Everything LiteLLM was trying to do — routing, translating, remapping — was solving for a mismatch that didn't exist anymore. The request already had the right name on it. The only thing left to do was answer.

With the direct connection verified, Chapter 7 gets Claude Code and Claude Desktop themselves actually pointed at it and doing real work.
