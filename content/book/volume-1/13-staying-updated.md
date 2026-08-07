---
title: "Chapter 13: Staying Updated"
weight: 13.0
draft: false
---

## The Tension This Chapter Actually Manages

Every piece of software in this book will eventually need updating — security patches, bug fixes, new features. Left entirely alone, a self-hosted stack quietly drifts out of date in a way a cloud service never does, because nobody's patching it for you. But update everything automatically and blindly, and you're one bad release away from a stack that was working yesterday and isn't today, discovered at the worst possible moment rather than on your own terms. This chapter is about managing that tension deliberately, rather than defaulting to either extreme.

## Watchtower, and Why Not the Obvious Image

Watchtower is the standard tool for this in a Docker-based stack — it watches your running containers, checks for newer versions of their images, and updates them automatically according to whatever policy you set. Nearly every guide you'll find online points you at the same image: `containrrr/watchtower`.

I don't run that one. I run `nickfedor/watchtower` instead — a maintained fork of the same project. Worth being honest about why, since it's a genuinely useful thing to know before you copy a Docker Compose file from somewhere and trust the image name without checking it: the original `containrrr/watchtower` project went quiet, without the ongoing maintenance a piece of infrastructure that auto-updates your other infrastructure really needs. Running an unmaintained tool to keep everything else maintained is exactly the kind of quiet contradiction that's easy to miss until it matters. The fork under `nickfedor` picked up active maintenance where the original left off, and that's the version I'd point you toward.

The setup itself is straightforward, added to the same Docker Compose file the rest of this stack has been living in:

```yaml
  watchtower:
    image: nickfedor/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=86400
    restart: unless-stopped
```

`WATCHTOWER_CLEANUP=true` removes old images after a successful update rather than letting them silently accumulate and eat disk space. `WATCHTOWER_POLL_INTERVAL=86400` checks once every 24 hours — frequent enough to stay current, infrequent enough that you're not hammering image registries or risking an update landing at an inconvenient moment multiple times a day.

## Update Hygiene: What Should Auto-Update, and What Shouldn't

This is the judgment call Watchtower's own defaults won't make for you, and it's worth thinking through deliberately rather than letting every container update itself blindly.

**Reasonable to auto-update:** nginx, Pipelines, Watchtower itself — well-established, actively maintained projects where a routine update is far more likely to be a genuine improvement or security fix than something that breaks your setup.

**Worth being more careful with:** anything where a version change could quietly alter behaviour you're depending on. Lemonade itself is the obvious one — an update that changes how it handles the aliasing from Chapter 5, or shifts default behaviour in a way that affects the direct connection from Chapter 6, is exactly the kind of thing you want to review before it lands automatically at 3am.

SearXNG belongs in this second category too, and I'm correcting myself here rather than quietly editing an earlier draft, because I initially had it filed alongside nginx as safe to auto-update, and that turned out to be wrong in a way that cost real debugging time. A routine SearXNG image update silently changed its bot-detection defaults, and the result was exactly the limiter problem Chapter 9 now covers — search that had been working stopped working, with no error, no crash, just requests that hung forever, because a new image version enabled a stricter default against a config file Watchtower has no reason to touch. Nothing about that failure looked like an update problem from the outside. It looked like a network issue, or Vane being broken, and it took checking SearXNG's own container logs specifically to find the actual cause.

The honest lesson isn't "never auto-update SearXNG" — it's that "supporting infrastructure" and "safe to auto-update" aren't actually the same category, and I'd conflated them. A service can be genuinely peripheral to your stack's core purpose and still have defaults that change in a way that breaks something real. If you're running SearXNG with Watchtower, either exclude it from automatic updates the same way this chapter already recommends for Lemonade, or make a habit of checking `settings.yml` after any update lands — specifically the `limiter` setting — rather than assuming an update to a supporting service is low-risk by default.

I'd suggest excluding Lemonade from Watchtower's automatic scope and updating it deliberately, on your own schedule, after checking release notes — a small extra bit of manual discipline for the pieces of this stack where an unreviewed change has the most potential to break something you'd notice immediately and have to debug from scratch.

Watchtower supports excluding specific containers from automatic updates with a label, if you want everything else on autopilot but Lemonade held back for manual review:

```yaml
  lemonade:
    labels:
      - "com.centurylinklabs.watchtower.enable=false"
```

## Rollback Strategy

However careful your update policy, something will eventually break — that's not pessimism, it's just the honest baseline for running your own infrastructure rather than someone else's managed service. The practical defence is simple and worth having in place before you need it rather than after: know how to roll a container back to the previous working image, quickly, without having to research the command under pressure.

```powershell
docker compose pull <service> --quiet
docker compose up -d <service>
```

pulls and restarts a specific service on its current pinned version. If you've let something float to `latest` and a new release breaks it, pinning to a specific previous tag in your Compose file and re-running `docker compose up -d` gets you back to known-good quickly — which is exactly why it's worth knowing the last working version of anything critical, rather than only ever running `latest` and hoping.

## The Portainer Situation, Honestly

I ran Portainer as a management UI on top of all of this — a genuinely useful visual layer for seeing container status, logs, and resource usage without living entirely in the command line. At the time of writing, mine was still on Portainer v1, which was legacy at that point; I'd flagged it for replacement with the current Community Edition LTS release (`portainer/portainer-ce:lts`) and hadn't actually done the migration yet. Consistent with the honesty this book's tried to keep throughout Part 3 and now Part 4: that was a real gap, on my own list, not yet closed by the time I was writing this chapter. If you're setting this up fresh rather than inheriting an older install the way I was, just start on the current CE LTS image directly and skip the migration I owed myself at the time.

## What "Done" Looks Like

Watchtower running on the maintained fork, checking daily, cleaning up old images, with Lemonade specifically excluded from automatic updates and everything else left to update itself. A known rollback command you could run without looking it up if something breaks. And an honest accounting — for your own build, not necessarily written down the way this chapter has been — of anything you're knowingly running on a legacy version, the way I was with Portainer at the time, so it stays a deliberate, tracked decision rather than something you've simply forgotten about.

Chapter 14 covers backups with Kopia — including the one step in a backup strategy that's easiest to skip and most important not to.
