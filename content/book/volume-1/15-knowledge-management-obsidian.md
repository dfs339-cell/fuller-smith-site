---
title: "Chapter 15: Knowledge Management with Obsidian"
weight: 15.0
draft: false
---

## Why a Notes App Belongs in a Book About AI Infrastructure

Everything so far in this book has been about the AI itself — hardware, models, the server, the tools that talk to it. This chapter is about something that sounds like a step backward at first: a plain, local, markdown-based notes app. The reason it belongs here is the same reason everything else in this book exists — I wanted my own notes, project documentation, and accumulated knowledge to stay genuinely mine, searchable and usable by the AI stack I've built, without any of it living in someone else's cloud.

Obsidian fits that requirement almost by design. It stores everything as plain markdown files on your own disk — no proprietary format, no account required to use it, nothing that stops working if a company changes its pricing or shuts down a service five years from now. That durability matters more here than it might in a lighter-weight project, because a knowledge vault is exactly the kind of thing you're still adding to years from now, and the last thing you want is to discover the format it's locked into no longer has anywhere to go.

## Obsidian as the Vault

Setting up Obsidian itself is the least remarkable part of this chapter — install it, point it at a folder, and that folder becomes your vault, with every note a plain `.md` file inside it. If you're already using Obsidian, there's nothing to change here; this chapter is about connecting an existing or new vault into the rest of the stack, not about Obsidian's own setup, which its own documentation covers better than I need to here.

The part worth being deliberate about is where that vault physically lives relative to everything else in this book. I keep mine on the same machine running Lemonade and the Docker stack, specifically so the RAG integration below doesn't need to reach across a network to read it — a local vault, read by a local RAG pipeline, feeding a local model, keeping the entire chain on hardware you actually own at every step.

## RAG Search via Open WebUI

Retrieval-augmented generation — RAG — is the mechanism that lets a model answer questions using your own documents rather than just what it learned during training. Open WebUI has this built in: point it at a folder of documents, and it indexes their content so that a relevant chunk gets pulled into context automatically when a question seems related to something in your vault, without you needing to paste the content in manually every time.

Setting this up means adding your Obsidian vault's folder as a document source inside Open WebUI's admin settings — Workspace, then Knowledge, then pointing at your vault's folder path. Once indexed, a question asked through Open WebUI that touches on something covered in your notes should pull relevant context automatically, genuinely changing the character of the answers you get back: specific to your own accumulated knowledge, not just general training data.

## The Honest Gap: Automated Ingestion

Here's where I need to be straightforward again, in keeping with the rest of this book: the *automated* side of this — a pipeline that notices when a note in the vault changes and re-indexes it without me doing anything manually — wasn't confirmed working in my own setup at the time. What I had was the basic integration: point Open WebUI at the vault, it indexes what's there. What I didn't yet have confidently verified was whether that index stayed current automatically as the vault changed, or whether it needed a manual re-index trigger I'd simply been doing without fully realising I was relying on it.

This matters practically: a RAG system quietly working against a stale index is a genuinely sneaky failure mode, similar in spirit to Chapter 14's untested backup — everything looks like it's working, right up until you ask about something you added last week and get an answer that doesn't reflect it, with nothing having told you the index was behind. If you're setting this up yourself, I'd suggest confirming explicitly, rather than assuming: edit a note, wait however long you'd expect an automatic re-index to take, then ask Open WebUI something that only the edited version would answer correctly. If it doesn't, you're in the same position I currently am, and a manual re-index step needs to become part of your actual routine until the automated side is properly confirmed.

## Claude Code and MCP: An Open Question at the Time

The other piece worth covering honestly rather than glossing over: giving Claude Code direct access to the Obsidian vault via MCP — the same protocol covered in more depth in the agent-harnesses article on the companion site — was something I looked into and didn't find a clean answer for at the time. There was no official Obsidian connector in the MCP registry as of my last check then, which meant this wasn't a five-minute setup the way pointing Open WebUI at a folder was.

The workarounds I considered rather than a connector that didn't exist yet: syncing the vault through a service Claude Code could already reach, or falling back to direct file upload for specific notes when they were actually relevant to what I was working on in Claude Code at that moment. Neither was as clean as a dedicated connector would have been, and I hadn't settled on one as a permanent answer by the time of writing this chapter. If an official MCP server for Obsidian exists by the time you're reading this — genuinely possible, given how fast the MCP ecosystem has been growing — that's very likely the better answer, and worth checking for before building around either workaround.

## What "Done" Looks Like, Honestly

A working Obsidian vault, read by Open WebUI's RAG integration, genuinely improving the relevance of answers for anything covered in your own notes — that part was solid. What wasn't yet solid, and what I'd ask you to verify properly rather than assume, the way I still needed to at the time: whether the index actually stayed current automatically, and whether Claude Code had any access to the vault at all beyond manual, one-off file sharing. Two real open questions, both flagged plainly rather than dressed up as finished.

Chapter 15b covers something Chapter 15 missed the first time round — getting non-markdown files into this same vault. Chapter 16 then covers security guardrails, including the guardrail layer question that's been sitting open since n8n came out of this stack.
