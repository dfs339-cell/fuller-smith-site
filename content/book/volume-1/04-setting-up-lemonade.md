---
title: "Chapter 4: Setting Up Lemonade"
weight: 4.0
draft: false
---

## Why Lemonade, Specifically

Before we install anything, a brief word on why this book builds around Lemonade rather than Ollama or a bare llama.cpp setup — both perfectly reasonable choices covered in more detail in Article 3 on the companion site, if you want the full comparison. The short version: Lemonade auto-detects your hardware and routes to the right backend without you managing driver-specific configuration yourself, and — the detail that actually matters most for this book — it can speak the OpenAI API, the Ollama API, and the Anthropic API simultaneously from the same server. That last one is the whole reason Chapters 5 through 7 work the way they do. Claude Code and Claude Desktop expect an Anthropic-shaped API. Lemonade already speaks it natively, which means the aliasing trick in the next chapter doesn't need any translation layer bolted on top.

If you're on hardware where Ollama makes more sense for you — straightforward Nvidia or Apple Silicon, no NPU, no mixed fleet — the model-aliasing concept in Chapter 5 still applies, you'll just be pointing a different server at it. But this book's walkthrough uses Lemonade throughout, so that's what we're installing now.

## Installing Lemonade Server

The cleanest install path on Windows is a single command, using `winget`, which ships built into Windows 10 (build 1809 and later) and Windows 11:

```powershell
winget install --id AMD.LemonadeServer --exact
```

This pulls the installer directly from AMD's own distribution and handles the setup for you, prompting for the license agreement along the way. If you'd rather have a visual installer instead of a terminal command, the GUI installer (`Lemonade_Server_Installer.exe`) is available from Lemonade's releases page and does the same job with click-through prompts, including letting you pick which models to install as part of setup.

Either way, once it's done, restart your terminal before doing anything else — a fresh PATH environment variable needs to be reloaded before the `lemonade-server` command will be recognised, and "command not found" immediately after installing is almost always just this, not a failed install.

Confirm it's actually there:

```powershell
lemonade-server status
```

## Starting the Server

By default, Lemonade Server runs on port 8000. I don't run it on the default port, and I'd suggest you don't either, for a small but genuinely useful reason: picking your own port makes it immediately obvious, at a glance, which service you're looking at when you've got several things running locally, rather than everything defaulting to the same handful of common ports and you having to check twice. My own setup runs on port 13305 — an arbitrary choice, nothing significant about the number itself, just one that's memorable to me and unlikely to collide with anything else on the machine.

To start the server on a custom port:

```powershell
lemonade-server serve --port 13305
```

Run that, and you should see the server start up, detect your available hardware, and report which backend it's selected — llama.cpp for GGUF models, FastFlowLM if it's found a supported NPU, and so on. This is worth actually reading rather than skimming past, because it's the first confirmation that Lemonade has correctly identified what you're running it on. If it reports CPU-only when you know you have a capable GPU, stop here and go back to Chapter 3's hardware verification steps before continuing — building on top of a misconfigured backend just means debugging the same problem later, with more layers stacked on top of it by then.

If you'd rather not have a terminal window open every time you want the server running, the Windows GUI application (`LemonadeServer.exe`) embeds the same server behind a system tray icon, and can be set to launch automatically at startup — genuinely worth doing once you trust the setup, so the server's simply always there when Claude Code or Desktop go looking for it.

## Keeping It Running: What Actually Starts Lemonade

I want to be honest about something here, because it's a small mistake worth you not repeating. Early on, I set up a Windows Task Scheduler entry — `LemonadeServerLoop` — pointing at a batch file meant to launch the server and keep it running through restarts. At some point since, that batch file stopped existing at the path the task still points to. The scheduled task is still there, still enabled, and as far as Task Scheduler is concerned it's a real, configured task. It just points at nothing.

I only found this out going back through my own setup to check the details for this chapter, which is exactly the kind of quiet gap this book has tried to be honest about elsewhere — Chapter 11's nginx, Chapter 13's Portainer. In this case, it didn't actually matter in practice, and it's worth explaining why rather than just flagging it as another loose end: Lemonade's tray application (`lemonade-app.exe`) and the server binary (`LemonadeServer.exe`) were already running independently of that scheduled task, launched via the tray app's own "start at login" setting rather than through Task Scheduler at all. The stale task wasn't doing anything, because it was never actually the thing keeping Lemonade alive — a genuine belt-and-braces setup that turned out to only have the belt.

The practical lesson: if you set up more than one mechanism to keep something running — a scheduled task, a tray app's own auto-start setting, a Windows service — periodically confirm which one is *actually* doing the job, the way I just did here, rather than assuming they're all still wired correctly just because none of them are complaining. A silently broken redundant mechanism gives you false confidence rather than genuine resilience.

Whichever mechanism you end up using, the way to check what's actually running Lemonade at any given moment is a straightforward process check:

```powershell
Get-Process | Where-Object {$_.ProcessName -like "*lemonade*"} | Select-Object ProcessName, Path, Id
```

If that returns both `lemonade-app` and `LemonadeServer`, the server's genuinely up, regardless of which mechanism actually started it.

## Loading Local Models

With the server running, the next step is pulling down actual models to serve. Lemonade's CLI handles this the same way Docker or Ollama would — a simple pull command against a name from its curated model list:

```powershell
lemonade-server pull <model-name>
```

The curated list covers most of the popular open-weight families in GGUF and ONNX format, with sizes and hardware suitability noted against each one — the same sizing homework Article 2 on the companion site walks through in detail, already partly done for you here. If a model you want isn't in the curated list, Lemonade's Model Manager (accessible once the server's running) can import custom GGUF and ONNX models directly from Hugging Face, which is where you'll end up if you're chasing something specific rather than working from the built-in catalogue.

I'd suggest starting with one small model to confirm the whole pipeline works end to end before pulling anything larger — there's no value in troubleshooting a multi-hour download of a large model if the actual problem turns out to be something earlier in the chain.

## Testing the Server Responds

Before moving on to Chapter 5's aliasing work, confirm the server is actually answering requests. The simplest check is the built-in web interface, which Lemonade serves directly:

```
http://localhost:13305/app
```

(Adjust the port to whatever you chose above.) You should get a working chat interface, letting you pick a loaded model and send it a message directly — genuinely the fastest way to confirm the whole stack, from hardware detection through to model response, is actually working before you introduce any other tooling into the mix.

For a more programmatic check — useful later when you're troubleshooting Claude Code's connection specifically — a direct API call confirms the same thing from the command line:

```powershell
curl http://localhost:13305/v1/models
```

That should return a JSON list of whatever models you've pulled and loaded. If it does, the server's genuinely ready, and the whole foundation the rest of Part 2 builds on is in place.

## What "Working" Actually Means Here

Resist the temptation to consider this step done the moment you get any response at all. Send a real message through the web interface — something with a bit of substance, not just "hello" — and actually read the reply. Confirm the response is coherent, arrives in a reasonable time for your hardware, and that the model you intended to load is the one that actually answered. I've had a Lemonade instance report a model as loaded when what was actually answering was a fallback default, and the only way I caught it was because the response style didn't match what I expected from the model I thought I'd loaded. A five-second sanity check here is cheaper than an hour of confused troubleshooting three chapters from now.

With the server running, a model loaded, and a confirmed response in hand, you're ready for the chapter that makes this book worth having bought: Chapter 5, where we make this server answer to Claude's own model names, and Claude Code stops knowing the difference.
