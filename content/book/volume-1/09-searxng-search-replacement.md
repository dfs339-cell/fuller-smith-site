---
title: "Chapter 9: SearXNG as a Search Replacement"
weight: 9.0
draft: false
---

## Finishing What Chapter 8 Left Open

Vane's been running since the last chapter, but without anything to actually search with — SearXNG is that missing piece, and wiring it in is what finally makes the friends track capable of answering questions about anything happening after your models' training data ends.

## Why SearXNG Over Tavily

Tavily, and services like it, are a perfectly reasonable choice if you want search-augmented AI answers without running anything yourself — pay per query or per month, get results back, done. I'd have used it without a second thought if this were a single, occasional-use setup. It isn't. This is infrastructure meant to run continuously, for more than one person, and two things about that changed the calculation for me.

**Cost at real usage volume.** A paid search API charges per query, and multiple people using search-augmented chat regularly adds up in a way that's easy to underestimate when you're only thinking about your own usage. Self-hosted search has a real setup cost in time, but no marginal cost per query afterward — the same tradeoff Chapter 1 walked through for the whole self-hosting decision, playing out again at a smaller scale, inside one specific piece of the stack.

**Privacy, extending the same principle the whole book is built on.** Every search query is, in a real sense, a piece of what someone's actually thinking about or working on. Routing that through a third-party paid API means that party sees every search your friends and family run through this system, indefinitely. SearXNG aggregates results from multiple underlying search engines without you or your users needing an account with any of them, and without your queries being logged by a service whose business model depends on knowing what people search for. For a system I'm asking other people to trust with their questions, that matters as much here as it did for the personal track's privacy argument in Chapter 1.

## Setting Up SearXNG

SearXNG runs as its own container, added to the same Docker Compose file from Chapter 8:

```yaml
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    ports:
      - "8080:8080"
    volumes:
      - ./searxng:/etc/searxng
    environment:
      - SEARXNG_BASE_URL=http://localhost:8080/
    restart: unless-stopped
```

The mounted volume (`./searxng:/etc/searxng`) gives you a local folder holding SearXNG's configuration — specifically `settings.yml`, which is where you'll actually control which underlying search engines it queries and how results get aggregated. On first run, SearXNG generates a default config into that folder if one doesn't already exist, which is the easiest way to get a working starting point rather than writing the config from scratch.

Bring it up the same way as before:

```powershell
docker compose up -d
```

And confirm it's reachable directly, before worrying about Vane's connection to it:

```
http://localhost:8080
```

You should get SearXNG's own search interface. Try a genuine query directly here first — this isolates whether SearXNG itself is working correctly, independent of whether Vane is successfully talking to it, the same "test each layer independently" principle Chapter 6 was built around.

## A Config Detail Worth Getting Right Early

The default `settings.yml` enables a broad set of search engines, not all of which you'll necessarily want. Some are slower, some are less reliable, and running searches against engines you don't actually care about just adds latency to every query for no benefit. It's worth opening the generated config and trimming the engine list down to a handful of ones you trust and actually want results from, rather than leaving every default enabled. This isn't strictly necessary to get things working, but it's the kind of thing that's much easier to do now, with no users depending on the system yet, than it is to revisit later once people are actively relying on it.

## The Limiter: A Setting That Will Silently Break Everything

This section exists because of a real outage, not a hypothetical one — SearXNG's default configuration ships with a `limiter` enabled, and it's built on an assumption this book's setup doesn't satisfy: that requests arriving at SearXNG are passing through a reverse proxy that sets `X-Forwarded-For` or `X-Real-IP` headers. In this book's setup, they aren't — you're hitting SearXNG directly, container to container or host to container, with nothing in between.

When the limiter is on and those headers are missing, SearXNG's bot-detection treats every request as suspicious. The failure mode is the worst kind: not an error, not a rejection, just a hang. Requests time out. Vane sits there waiting. Nothing in the logs points at the actual cause unless you go looking specifically for `X-Forwarded-For nor X-Real-IP header is set` warnings buried in SearXNG's own container logs. From the outside, it looks exactly like SearXNG itself is broken, or slow, or down — none of which is true.

This is a genuine risk on a fresh install, and it's an even bigger one after any SearXNG image update, since a version bump can silently enable stricter defaults on a config you haven't touched since Chapter 9 first generated it. If search that was working suddenly starts hanging on every query with no obvious cause, this is the first thing to check — before assuming a network problem, a Vane misconfiguration, or anything more exotic.

The fix, appropriate for this book's single-user, no-reverse-proxy setup, is to disable the limiter explicitly in `settings.yml`:

```yaml
server:
  limiter: false
```

Restart the container after the change:

```powershell
docker compose restart searxng
```

Then re-run the direct browser test from the previous section. If it was hanging before, it should now return results immediately.

## Wiring the Connection to Vane

If Chapter 8's Vane configuration is still in place, the connection is already pointed at the right address — `SEARXNG_API_URL=http://host.docker.internal:8080` — and it should simply start working now that something is actually listening there. Restart Vane's container to make sure it picks up a fresh connection rather than an old failed one it might be holding onto:

```powershell
docker compose restart vane
```

## The Real End-to-End Test

This is the point where the friends track becomes genuinely more capable than the personal track in one specific way: real-time information. Through Open WebUI, with Vane enabled, ask something that couldn't possibly be answered from a model's training data alone — something about very recent, current information. If the pipeline's correctly wired, you should see the response drawing on live search results, not just the model's own internal knowledge.

If it doesn't work, check the layers in order, the same troubleshooting discipline from Chapter 6: confirm SearXNG itself returns results when queried directly first. If that works but Vane still isn't using it, check Vane's own logs for connection errors — a `host.docker.internal` resolution problem is the most common culprit at this stage, the same networking detail Chapter 8 introduced.

## What "Done" Looks Like

Open WebUI, Vane, and SearXNG all running together, with a genuine search-augmented query returning results that clearly draw on current information rather than just the model's own training. That's the friends track's core capability actually working — not just a chat interface with a model behind it, but something meaningfully more useful than what most people are used to from a plain chatbot.

Chapter 10 looks at Pipelines — extending what this stack can do beyond chat and search.
