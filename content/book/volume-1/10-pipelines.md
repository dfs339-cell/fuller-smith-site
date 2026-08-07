---
title: "Chapter 10: Pipelines"
weight: 10.0
draft: false
---

## The Difference Between a Chatbot and a System

Everything built so far in Part 3 gives your friends and family a private, search-capable chat interface — genuinely useful on its own, and for a lot of households, that's a complete and satisfying stopping point. This chapter is for anyone who wants more than that: the ability to actually shape how the system behaves, not just which model answers.

Pipelines is Open WebUI's plugin framework — a separate, lightweight service that sits in the request chain between Open WebUI and Lemonade, letting you run custom logic against every message before it reaches a model, after a model responds, or both. I should be upfront about where this stood in my own build at the time, before going further: Pipelines wasn't something I ran as core, always-on infrastructure the way Open WebUI or SearXNG were. It was something I'd built one real, useful thing with, learned a genuinely useful lesson from, and hadn't extended much beyond that single use case. This chapter is honest about that rather than presenting Pipelines as a bigger part of the stack than it actually was for me at the time.

## What I Actually Built: A Guardrail Pipeline

The problem I wanted to solve was straightforward: before anything gets sent to a model — especially on the friends track, where I don't control what someone else pastes in — I wanted a check for sensitive data. Email addresses, names, company details, the kind of thing that's easy to paste into a chat window without thinking about it, especially if someone's dropping in a document rather than typing a question from scratch.

**The first attempt blocked outright.** The pipeline scanned inbound content, and if it matched a pattern for an email address, a name, or a known company identifier, it rejected the request entirely and returned an error. This seemed like the safe, conservative choice going in. In practice, it fell apart the moment someone tried to paste in an actual document rather than a short question — a real email, a real set of meeting notes, anything with genuine substance almost always contains at least one of the things the filter was watching for, and the entire document would bounce with an error message instead of being usable at all. A guardrail that blocks most real work isn't a guardrail anyone's going to tolerate for long — they'll route around it, or stop using the system, neither of which is the outcome I wanted.

**The fix was to redact instead of block.** Rather than rejecting content that matched a sensitive-data pattern, the revised pipeline strips or masks the matched portions and lets the rest of the message through — an email address becomes `[REDACTED EMAIL]`, a recognised name or company gets masked the same way, and the model receives a version of the content with the sensitive specifics removed but the actual substance intact. This is genuinely working well now, and it's the version I'd point you toward if you're building the same thing rather than the block-first version I started with.

The lesson underneath the specific fix, worth taking with you into any guardrail you build yourself: a filter that fails safe by blocking everything that trips it isn't actually a safe default if it makes the system unusable — it just pushes people toward working around it entirely. Redacting and passing through is more work to build correctly, but it's the version that actually gets used.

## Setting Up Pipelines

Pipelines runs as its own container, added to the same Docker Compose file that's been growing throughout Part 3:

```yaml
  pipelines:
    image: ghcr.io/open-webui/pipelines:main
    container_name: pipelines
    ports:
      - "9099:9099"
    volumes:
      - ./pipelines:/app/pipelines
    restart: unless-stopped
```

The mounted volume is where your actual pipeline scripts live — drop a Python file implementing Pipelines' expected interface into that folder, and it becomes available to enable from within Open WebUI's admin settings without needing to rebuild or restart the container each time.

Once it's running, connect Open WebUI to it: in Open WebUI's admin settings, under Connections, add Pipelines as an additional OpenAI-compatible endpoint, pointing at `http://host.docker.internal:9099`, using the same placeholder-key pattern from every other service in this book so far. Once connected, any pipeline scripts you've added become selectable, the same way a model would be.

## A Worked Example: Redact, Don't Block

Here's the shape of the working version, simplified to the core pattern rather than the full implementation:

```python
import re
from pydantic import BaseModel

class Pipeline:
    class Valves(BaseModel):
        pass

    def __init__(self):
        self.name = "Sensitive Data Redactor"
        self.email_pattern = re.compile(r'[\w\.-]+@[\w\.-]+\.\w+')
        # Extend with your own patterns or a name/company lookup list

    async def inlet(self, body: dict, user: dict) -> dict:
        last_message = body['messages'][-1]['content']
        redacted = self.email_pattern.sub('[REDACTED EMAIL]', last_message)
        # Apply additional redaction rules here — names, company identifiers, etc.
        body['messages'][-1]['content'] = redacted
        return body
```

This is deliberately the simple version — a single regex pattern for email addresses, redacting rather than rejecting. Names and company details need a different approach than a regex can reasonably handle well; a maintained lookup list of names and terms you actually want caught gets you further than trying to pattern-match names generically, which produces too many false positives and misses to be useful on its own. Start with the pattern-matchable cases like email addresses, confirm redact-not-block is working the way you want, then extend carefully rather than trying to catch everything on the first attempt — which is exactly the mistake the block-first version made in a different way.

## Is This Chapter Optional?

For most setups, genuinely yes. If Open WebUI, Vane, and SearXNG from Chapters 8 and 9 already give the people you're building this for everything they need, there's no obligation to add Pipelines just because it's available. It earns its place specifically if you're handling content sensitive enough to want an inbound guardrail, the way I was — and if you do build one, take the lesson from this chapter for free rather than learning it the way I did: redact, don't block, or you'll build something technically safe that nobody actually wants to use.

Chapter 11 covers nginx and routing — the piece that makes this entire friends-track stack reachable cleanly, by more than one person, without anyone needing to remember which port does what.
