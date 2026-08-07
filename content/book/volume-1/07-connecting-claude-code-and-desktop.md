---
title: "Chapter 7: Connecting Claude Code and Claude Desktop"
weight: 7.0
draft: false
---

## The Anticlimactic Chapter, By Design

After Chapter 6's two-day detour through LiteLLM, I promised this chapter would be almost anticlimactic. That's genuinely the goal. If Chapters 4 through 6 are properly wired, actually connecting the client tools should feel like nothing much happens — you open them, they work, and the interesting engineering is already behind you. This chapter is mostly about confirming that's actually true, and knowing where to look on the rare occasion it isn't.

## Claude Code

With the environment variables from Chapter 6 set — `ANTHROPIC_BASE_URL` and `ANTHROPIC_API_KEY` — open a fresh terminal (fresh matters here; an already-open terminal from before you set those variables won't have picked them up) and start Claude Code as normal.

The first genuine sanity check isn't a special command — it's just asking it something. Send a real message, something with enough substance that you can tell which model actually answered, and read the reply the way Chapter 4 asked you to read Lemonade's own web interface response: not just "did something come back," but "does this response's character match the model I aliased to this tier." If you aliased a coding-tuned model to your Sonnet tier, ask it something with real coding substance and see if the answer reflects that.

If Claude Code responds but the response feels off — generic in a way that doesn't match what you know about your aliased model, or suspiciously fast for hardware you know isn't that fast — stop and check whether it's actually hitting your local server at all, rather than assuming the aliasing is at fault. Environment variables that didn't load because of a stale terminal session are the most common cause of this, and the symptom looks like a model problem when it's actually a "still talking to Anthropic's cloud" problem.

## Claude Desktop: Code and Cowork Tabs Specifically

This is worth being precise about, because it trips people up: plain Claude Desktop chat — the default window you get on opening the app — doesn't support a custom backend at all. That's true by design, not a gap in this book's instructions. The capability you actually want lives in two specific places within Desktop: the Code tab, which behaves the same way the standalone Claude Code CLI does, and the Cowork tab, which reaches the same local server through a feature Anthropic documents as "Cowork on 3P" (third-party inference).

**For the Code tab**, the same environment variables from Chapter 6 apply directly — if Claude Code CLI is working, the Code tab inside Desktop should pick up the identical configuration without anything further needed.

**For Cowork**, there's an explicit step: third-party inference needs to be turned on in Cowork's own settings before it'll route anywhere other than Anthropic's cloud, even with the right environment variables set. It's a deliberate opt-in, not a bug if it doesn't work the moment you set the variables and expect it to just apply — go into Cowork's settings and confirm third-party inference is actually enabled, pointing at the same local endpoint.

**One limitation worth knowing before you rely on it:** in third-party inference mode, Anthropic's own Connectors — Slack, Google Drive, and similar first-party integrations — go unavailable. They depend on Anthropic's own infrastructure specifically, not just the model, so pointing Cowork at a local model doesn't carry them along. If you need that kind of tool access while running locally, MCP servers are the documented route back to equivalent functionality, and it's worth planning for that rather than being surprised mid-task that a Connector you were relying on has quietly disappeared.

## A First Real Task, Not Just a Test Message

Once both tools are responding and you've confirmed the character of the response matches what you expect, give the setup one genuine piece of real work before considering this chapter done — not a synthetic "say hello" test, but something you'd actually use it for. A real coding question if you aliased a coding model to the tier you're testing, a real piece of drafting if that's more your use case. The reason this matters more than it might seem: synthetic tests can pass while something subtler is still wrong — a model responding coherently to a trivial prompt doesn't guarantee it'll hold up on the kind of task you actually bought this whole setup to do.

## Troubleshooting at the Client Layer

Chapter 6 covered troubleshooting the connection itself — the server, the aliasing, the direct-connection config. This is a different, narrower set of things that can go wrong specifically at the client layer, once the config is already proven to work in isolation.

**Stale terminal sessions.** Covered above, worth repeating because it's genuinely the most common cause: any terminal or app instance open before you set the environment variables won't see them. Close and reopen, every time, after a config change.

**Desktop needs a full restart, not just a new tab.** Opening a new Code or Cowork tab in an already-running Desktop instance doesn't necessarily re-read environment variables set since the app launched. If a fresh terminal-based Claude Code works but Desktop doesn't, fully quit and relaunch Desktop before assuming something's actually broken.

**Cowork's third-party inference toggle resets on updates, occasionally.** Worth a quick check after any Desktop update if Cowork suddenly seems to be back on Anthropic's cloud unexpectedly — settings toggles are exactly the kind of thing that can get reset by an update you didn't think would touch your configuration.

**Confirm which tab you're actually in.** This sounds almost too obvious to mention, but it's caught me out: Desktop's plain chat window, the Code tab, and the Cowork tab are three different surfaces with three different capabilities, and it's easy to be troubleshooting a "why won't this connect locally" problem in the one tab that was never going to support it in the first place. Check you're actually in Code or Cowork before troubleshooting further.

## What "Done" Looks Like for Part 2

With Claude Code, Desktop's Code tab, and Cowork all confirmed working against your local server — each one answering with the character you expect from your aliased models, each one having handled at least one genuine piece of real work rather than just a test message — Part 2 of this book is complete. You have a fully private, fully local AI assistant, wired into the tools you'd actually use day to day, with no cloud dependency in the loop unless you deliberately choose one.

Part 3 turns to the friends track — extending this same underlying capability to other people, without asking any of them to touch a terminal or an environment variable.
