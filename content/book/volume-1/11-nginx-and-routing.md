---
title: "Chapter 11: nginx and Routing"
weight: 11.0
draft: false
---

## A Gap I'll Be Honest About Upfront

I want to start this chapter the same way I started the last one: with where my own build actually stood at the time, not a tidier version of it. nginx wasn't running in my stack yet, for the routing purpose this chapter is about to walk through. I knew it was needed — genuinely, not as a hedge — and I'll explain exactly why over the next few pages. But I'd rather tell you that plainly than write this chapter as though it was already solved, when what I actually had at the time was Open WebUI, Vane, SearXNG, and Pipelines each sitting on their own port, reachable individually, with nothing yet tying them into the single clean entry point they genuinely deserved.

That gap is worth naming rather than glossing over, because it's a useful illustration of something true about a project like this: the personal track, built for one trusted user, was comfortable to leave minimal. The friends track, the moment it's actually serving other people, doesn't get to stay that way — and admitting I haven't closed this particular gap yet is more useful to you than pretending every chapter in this book represents a finished, polished piece of my own infrastructure.

## Why the Friends Track Needs This (and the Personal Track Doesn't)

Chapter 2 introduced this distinction in outline; here's the concrete version. Right now, reaching the friends track stack means knowing that Open WebUI lives on port 3000, Vane on 3030, SearXNG on 8080, and Pipelines on 9099 — four different addresses, for what should feel like one coherent thing to anyone who isn't you. That's a fine arrangement for a technical single user who already knows the map. It's a genuinely bad experience for family or friends who just want to open one link and have it work, and it only gets worse if you ever want to add another service to the stack without asking everyone to memorise a new port number.

nginx solves this by sitting in front of everything as a reverse proxy — one address, routing internally to whichever backend service actually handles a given request, invisible to whoever's using it. The personal track never needed this because it was never more than one trusted user talking directly to one server. The moment "one trusted user" becomes "several people who shouldn't need to think about ports," a router genuinely earns its place, in a way nothing in Part 2 required.

## The Basic Reverse Proxy Config

Here's the shape of what this needed to look like, even though — to be honest one more time — I hadn't run this exact config in production myself at the time of writing. This was the plan, built the same way every other config in this book has been: reasoned through properly, ready to deploy, and worth you testing carefully rather than assuming it's battle-tested the way Chapters 4 through 9 were by the time I wrote them.

nginx itself can run as another container in the same Docker Compose stack:

```yaml
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - open-webui
      - vane
      - searxng
    restart: unless-stopped
```

And the actual routing configuration, mounted in from `./nginx/nginx.conf`:

```nginx
events {}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://open-webui:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /search/ {
            proxy_pass http://searxng:8080/;
            proxy_set_header Host $host;
        }
    }
}
```

The pattern worth understanding here, not just copying: `location` blocks match on the path someone's actually requesting, and `proxy_pass` sends that request on to the right internal container — using Docker's own service names (`open-webui`, `searxng`) rather than `host.docker.internal`, since nginx and these other services are all containers on the same Docker network talking to each other directly, not reaching back out to the host machine the way Chapter 8's Vane-to-Lemonade connection needed to. Everything under `/` goes to Open WebUI, which is the main interface everyone will actually use. A path like `/search/` can expose SearXNG's own interface directly, if you want that reachable separately from the main chat experience — genuinely optional, and easy to remove from the config if you'd rather keep everything strictly behind the main interface.

## Routing Multiple Services Cleanly

The principle that scales past just these two services: every new thing you add to the friends track gets its own `location` block, routed to its own container, all reachable through the same single address on port 80. Someone using this system never needs to know Vane exists on its own port, or that Pipelines is a separate service entirely — they open one address, and nginx quietly sorts out which container actually answers.

This is also where a genuinely non-technical user's experience starts to feel like a real product rather than a home lab experiment. A single bookmarked address, on one port, that just works — that's the bar this chapter is aiming at, and it's a meaningfully different experience from the four-separate-ports reality my own build is still sitting at while I write this.

## What I'd Actually Do Next, If I Were You

Don't leave this the way I did at the time. If you're building the friends track for people who genuinely aren't going to remember four port numbers — and if you're building it at all, that's almost certainly who it's for — treat this chapter as required, not optional, even though I've just told you it was the one part of my own build still sitting undone when I wrote this. Test the config above against your own running services from Chapters 8 through 10, confirm each `location` block actually reaches the right container, and close this gap properly before real people start relying on the system. I closed it myself shortly after writing this chapter, for exactly the reason just given.

Chapter 12 covers Tailscale Serve — getting this whole stack, once it's properly routed, reachable securely from outside your own network.
