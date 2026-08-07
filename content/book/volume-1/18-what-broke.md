---
title: "Chapter 18: What Broke"
weight: 18.0
draft: false
---

## Why This Chapter Exists

This chapter documents failures—not just gaps in completion, but real breakdowns that consumed time and revealed critical misconceptions. Unlike earlier chapters, it doesn’t just describe open problems; it tells what went wrong *badly enough* to interrupt work, and what I’d do differently if starting over. Two of these incidents are retold from earlier chapters; four are new.

## Incident One: Two Days Fighting the Wrong Layer

**What happened:**  
Reached for [[LiteLLM]] as a gateway between [[Claude Code]] and [[Lemonade]], assuming it was the “correct” solution for routing between API providers. After two days of misconfiguration and failed tests, discovered LiteLLM’s core purpose—model name remapping—*broke* the aliasing trick from [[Chapter 5]], which relied on transparent, unchanged endpoint routing.

**Root cause:**  
Assumed a “properly engineered” standard tool must fit the problem, without verifying whether the same problem had already been solved elsewhere in the stack.

**What I’d do differently:**  
Before adopting any external tool, explicitly ask: *Has this specific problem already been solved at a different layer?* Don’t default to recognized standards without validating fit.

## Incident Two: A Guardrail That Blocked Real Work

**What happened:**  
Implemented a content filter (Chapter 10) to block sensitive data (emails, names, company info) before reaching models. First version *rejected* any document matching the criteria—seemingly safer, but rendered the system unusable with real-world documents, since genuine files almost always contain sensitive data.

**Root cause:**  
Designed the guardrail around *blocking* (a blunt instrument) rather than *handling flagged content appropriately* (e.g., masking, transforming).

**What I’d do differently:**  
Build guardrails around what should *happen* to flagged content—not just whether to allow it. Prefer redaction over rejection to preserve utility while protecting privacy.

## Incident Three: The Memory Math I Got Wrong

**What happened:**  
Initial setup of the three-tier aliasing system (Chapter 5) assumed sizing based *only on model weights*. Worked fine in light testing but failed under real load—large code files, multi-turn conversations—slowing or crashing the inference server due to unaccounted [[KV cache]] memory usage.

**Root cause:**  
Overlooked that context size scales *on top* of weights, especially in high-usage scenarios. Mistook “weights fit” for “total workload fits.”

**What I’d do differently:**  
- Size for *context budget per tier*, not just model size.  
- Test with heavy, realistic usage *before* trusting setup—not just “hello” responses.  
- Assign models to tiers based not only on capability, but on realistic context load capacity.

## Incident Four: Two Failures on the Same Afternoon, Same Root Cause

**What happened:**  
Two seemingly unrelated failures on the same afternoon:  
- [[SearXNG]] hung on every query (no error, just infinite wait).  
- [[Vane]] crashed or echoed tool-call syntax instead of answering when search results were included.

**Root causes:**  
- SearXNG: Silent update enabled stricter bot-detection; no reverse proxy set required headers.  
- Vane: Ran on a model with insufficient context and unreliable tool-calling—fine for one-liners, failed under real search synthesis.

**Root cause *shared*:**  
Both layers passed light testing (manual query, simple prompt) but were never stress-tested under *actual sustained or complex usage*. Assumed “working in demo” meant “robust for real.”

**What I’d do differently:**  
Test layers *before deployment* under conditions matching expected load. Avoid the illusion of safety from light testing—it can falsely confirm stability.

## Incident Five: Two Blind Spots, One Old Model Name

**What happened:**  
Two separate failures weeks apart, both caused by *unmanaged references* to the same unaliased model name:  
1. Overnight research pipeline crashed due to memory allocation failures. Logs showed a stray, unaliased model loaded alongside the intended three-tier setup—leftovers from earlier experiments still in the model folder.  
2. A compiler script, written months earlier, still referenced the raw coder model by name. When it failed on context size, the failed request was re-enqueued repeatedly, blocking the intake folder.

**Root cause:**  
A single stale model reference was *copied or hand-typed into multiple scripts over time*, with no process to audit or deprecate obsolete references. Cleanup of one failure didn’t find the other.

**What I’d do differently:**  
- Treat model folders as *declared environments*, not open directories—limit what can be loaded.  
- Enforce model registration, versioning, and deprecation *explicitly*.  
- After any fix: *Where else might this same bad assumption be hiding?* Don’t assume one fix cleans all instances.

## Incident Six: PowerShell's Own Silent Failure Modes

**What happened:**
Building the Crawl4AI fallback described in [[Chapter 17]] surfaced four separate Windows PowerShell issues, none related to Crawl4AI itself, each discovered live rather than anticipated in advance. A downloaded `.ps1` file silently refused to dot-source under `RemoteSigned` policy because of a Mark-of-the-Web flag, with an old in-memory function definition masking the fact that edits on disk were never actually loading. `Invoke-RestMethod` and `Invoke-WebRequest` mangled UTF-8 into mojibake whenever a server didn't declare its charset explicitly. `-TimeoutSec` failed to fire against a genuinely hung connection — a Lemonade call sat frozen at 0% CPU for several minutes past its configured 180-second timeout with no error ever surfacing. And the overnight pipeline's seen-list was only being persisted once, at the end of a run, so an RDP session dropping mid-run — the idle-disconnect policy kills the process instantly — silently discarded a full run's completed work.

**Root cause:**
Trusting PowerShell's own tooling to behave the way its parameters and defaults imply, rather than verifying that behaviour under the specific conditions this pipeline actually runs in — unattended, long-running, over RDP, against servers with inconsistent charset headers. Each of the four issues is a case where the documented behaviour and the actual behaviour quietly diverged, and none of them produced an error that pointed at the real cause.

**What I'd do differently:**
- Unblock every downloaded script explicitly with `Unblock-File` before trusting that edits on disk are the code actually running.
- Decode HTTP responses explicitly with `[System.Text.Encoding]::UTF8.GetString()` rather than trusting automatic charset detection.
- Enforce timeouts from outside a call — `Start-Job` plus `Wait-Job -Timeout N` — rather than trusting `-TimeoutSec` against a connection that can hang without erroring.
- Wrap unattended loops in `try/finally` and persist state after every item, not once at the end, since anything long-running and unattended will eventually get killed mid-run.

## The Pattern Underneath All Six

Every failure shared the same core issue:  
**The gap between *light testing* and *real-world load*.**  
- Light testing confirmed *technical response*, not *robust behavior*.  
- Real usage revealed failures *weeks or months* after deployment—often only when under actual, complex, or sustained load.

This isn’t about tools being bad. It’s about *how* they’re used—and *how thoroughly* they’re stress-tested.

### Key Insight  
> Fixing the visible failure doesn’t mean you’ve found every instance of the same mistake. Stale assumptions persist in blind spots—often in multiple scripts, configs, or systems.

### Practical Advice  
1. **Test under real load before trusting.** Not just “does it respond?” but “does it handle *my* workload, *my* data, *my* usage patterns?”  
2. **Make failure loud.** Prefer clear, immediate errors over silent degradation—especially for unattended systems.

## Advice for Starting From Scratch Today

If I were rebuilding this from zero:  
- **Build Part 2 first (the personal track)**—and *use it for real work* for weeks before touching Part 3.  
  - Most failures happened in automation/maintenance layers because they lacked daily, real-world validation.  
- **Test every layer *as if it will be used heavily*.** Even if it’s for “other people” or “background tasks.”  
- **Restore your backup *before you need it*.** A backup you’ve never restored is a belief, not a fact.

## Where This Leaves You

Twenty chapters, counting the ones that only earned a "b." Two tracks. A handful of things still unfinished—not hidden, but declared honestly. This is not a polished sales pitch. It’s the *real* state of a working system, built incrementally and imperfectly.

The Sovereign AI Lab runs daily, delivers real value, and taught me more than any documentation could—including what’s broken, and why. If you’ve built along with this, you have your *own* version now: private, yours, on hardware you own—answerable to no one.

Whatever you build next is genuinely up to you. Thank you for reading this far—and good luck with the build.