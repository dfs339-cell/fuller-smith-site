---
title: "Chapter 8: Open WebUI + Vane/Perplexica Setup"
weight: 8.0
draft: false
---

## Starting Part 3 Properly

Part 2 was about you — a direct, single-user connection between tools you already know how to use and a server only you need to understand. Part 3 is about everyone else. The moment you're building for someone who isn't going to open a terminal, doesn't want to hear about environment variables, and just wants something that looks and feels like the AI chat interfaces they already use, the whole approach has to change. That's what this chapter starts building.

## Why Open WebUI

Open WebUI is the front end for the friends track, and the reason is straightforward: it looks and behaves like the commercial AI chat interfaces most people already have some familiarity with, it's genuinely well-maintained, and — the detail that matters most for this book — it speaks the OpenAI API format natively. That's a different requirement from Part 2's tools. Claude Code and Desktop needed an Anthropic-shaped endpoint, which is why Chapters 5 and 6 were built around aliasing to Claude's model names. Open WebUI needs an OpenAI-shaped one instead, and Lemonade — as established back in Chapter 4 — serves both simultaneously from the same install. You're not standing up a second inference server for the friends track. You're pointing a second front end at the same one.

## Installing Open WebUI

Open WebUI runs as a Docker container, which is exactly why Chapter 3 had you get Docker Desktop properly configured before any of this started. The simplest install is a single Docker Compose service:

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    environment:
      - OPENAI_API_BASE_URL=http://host.docker.internal:13305/v1
      - OPENAI_API_KEY=local-lemonade-no-key-needed
    volumes:
      - open-webui-data:/app/backend/data
    restart: unless-stopped

volumes:
  open-webui-data:
```

A few details worth understanding rather than just copying blindly. `host.docker.internal` is Docker Desktop's way of letting a container reach back out to services running on your actual Windows machine — since Lemonade is running natively on your host, not inside a container itself, the WebUI container needs this special hostname to find it, rather than `localhost`, which inside a container would refer to the container itself. The port mapping — `3000:8080` — means Open WebUI's internal port 8080 is exposed on your host machine as port 3000, which is the address you'll actually browse to. And the same placeholder API key pattern from Chapter 6 applies here too: Lemonade isn't validating it, but Open WebUI expects the field present.

Bring it up with:

```powershell
docker compose up -d
```

Then browse to `http://localhost:3000`. The first visit prompts you to create an admin account — do this before anyone else touches the system, since the first account created becomes the administrator by default, with the ability to manage every other user who signs up afterward.

Once you're in, confirm the connection to Lemonade actually works: Open WebUI's model selector should show your available models, including your aliased ones from Chapter 5 if you left them registered under both names. Send a genuine test message the same way Chapter 4 and Chapter 7 both asked you to — read the response, don't just confirm one arrived.

## Adding Vane

Open WebUI on its own is a solid chat interface, but it doesn't give your users real-time web search or browsing out of the box — for that, this build adds Vane, running as a companion service alongside it. (Perplexica is a reasonable alternative here, covering similar ground; I settled on Vane for my own setup, and this chapter documents that choice, but the two are close enough in purpose that swapping one for the other doesn't change anything else in this book.)

Vane runs as its own container, added to the same Docker Compose file:

```yaml
  vane:
    image: vane/vane:slim
    container_name: vane
    ports:
      - "3030:3000"
    environment:
      - SEARXNG_API_URL=http://host.docker.internal:8080
    depends_on:
      - open-webui
    restart: unless-stopped
```

Note the `slim` image tag specifically — this is a deliberately lighter build, skipping bundled extras you don't need when you're already running Open WebUI as your primary interface and just want Vane's search-augmentation capability alongside it, not a second full chat interface competing for the same job.

You'll notice the `SEARXNG_API_URL` pointing at port 8080 — that's SearXNG, the private search backend Vane needs to actually do anything useful, and it's not running yet. That's deliberately the next chapter's job, not this one's. Bring this configuration up now regardless; Vane will start, but search-dependent features won't work correctly until Chapter 9 is done. I've structured it this way on purpose — get the container running and confirmed healthy first, then wire in what it depends on, rather than debugging two new services introduced at once.

Confirm Vane's at least running:

```powershell
docker compose ps
```

You should see both `open-webui` and `vane` listed as up. A genuine end-to-end test of Vane's actual search capability waits until Chapter 9 gives it something to search with.

## The Model Selection Trap

One thing worth doing now, even though it won't matter until Chapter 9's search test, because it's easy to miss: Vane's own Settings — inside the running app itself, under Models, not the Docker Compose environment variables — has its own chat model dropdown, and it does not necessarily default to whatever `CHAT_MODEL` you set in the container's environment. Vane persists its own selection in its internal database, and that stored choice takes priority over the env var on every subsequent start.

This matters because Vane's actual workload — taking a batch of real search results and synthesizing them into an answer with citations — needs two things a lightweight model tier often doesn't have: enough context window to hold several full search results at once without truncating, and reliable tool-calling support for however Vane invokes search internally. Get either wrong and the failure mode isn't a clean error. It's a context-overflow exception on a real query that worked fine on a simple test message, or a response that echoes raw tool-call syntax back at you instead of a real answer — both of which look like Vane itself is broken when the actual problem is just which model it's quietly still pointed at.

Before trusting Vane's search integration in Chapter 9, open Vane's own Settings → Models page and confirm explicitly which chat model is selected — don't assume the env var took. Point it at whichever of your aliased tiers has the largest context window and confirmed tool-calling support, not necessarily the fastest or lightest one.

## What "Done" Looks Like for This Chapter

By the end of this chapter, you should have Open WebUI running at `http://localhost:3000`, an admin account created, a confirmed working connection through to Lemonade with a real test message answered correctly, and Vane running alongside it — even if not yet fully functional, since it's still waiting on SearXNG. That's a genuine milestone: the first piece of infrastructure in this book built for someone other than you, up and reachable, even before the rest of Part 3 finishes wiring it together properly.

Chapter 9 gives Vane something to actually search with.
