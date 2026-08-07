---
title: "Chapter 15b: The Ingestion Gap I Missed"
weight: 15.5
draft: false
---

## Why This Is a "b" Chapter, Not a Rewrite

This one exists because I got the first draft of this book wrong, in a specific and honest way worth naming rather than quietly fixing. Chapter 15 covers the vault, and it covers how Open WebUI's RAG integration reads it — but it never actually explains how anything other than a markdown file gets *into* that vault in the first place. Every example in Chapter 15 assumes the content already exists as a `.md` file sitting in a folder. Most of what I actually wanted in the vault didn't start that way. It started as a PDF, a PowerPoint deck, a Word document, or an Excel sheet, downloaded or received from somewhere else entirely.

Rather than rewrite Chapter 15 and pretend I'd planned for this from the start, I'd rather add this as its own chapter, exactly where it belongs chronologically and honestly: a gap in the original build that I only noticed because I kept hitting it in practice, not because I designed around it up front.

## What Started It: A File the Original Script Couldn't Use

The first version of this wasn't the multi-script setup described below — it was a single script, built to watch for plain text files landing in Downloads and drop them straight into the vault as markdown notes. For a while, that was genuinely enough. Most of what I was downloading and wanted indexed was already text-shaped, and a simple watch-and-copy script handled it without any real complexity.

It kept working right up until it didn't. A PDF landed in Downloads, the script picked it up the same way it picked up everything else, and what it actually wrote into the vault was unusable — a PDF isn't plain text, and treating it like one produced a note full of binary noise rather than anything a model could meaningfully read. A PowerPoint file a little while later hit the same wall for a different reason: PPT files are a completely different internal structure, not text at all in the way the original script assumed everything would be.

Neither failure was loud. Nothing crashed. A markdown file appeared in the vault either way — it just wasn't a useful one, which is a quieter and in some ways more dangerous failure than an outright error, because nothing told me it had happened. I only noticed because I went looking for something I knew I'd downloaded and the vault's answer about it made no sense.

## The Assumption I Started With, and Why It Was Wrong

Once the PDF and PowerPoint failures made it obvious the original text-only script couldn't just be left as-is, my first instinct was to extend it — teach the same script to detect a file's type and branch into different extraction logic depending on what it found, rather than throw it out and start over. One script, a bit more logic inside it, still one entry point. That felt like the natural next step rather than a redesign.

It didn't hold up past the second format I added. PDF extraction, PowerPoint extraction, Word extraction, and Excel extraction are genuinely different problems wearing the same "give me the text out of this file" description. A PDF might be text-based or it might be scanned images with no text layer at all. A PowerPoint's content lives in slides and speaker notes, not a single linear text stream. Excel is structured data, not prose, and flattening a spreadsheet into markdown paragraphs loses exactly the structure that made it useful in the first place. Trying to handle all of that inside one growing script meant a tangle of format-detection logic and conditional branches that got harder to trust with every new type I added, and a bug in one format's handling risked quietly breaking another's — including, ironically, the plain-text path that had been working fine from the start.

The lesson, worth taking with you if you're tempted to do the same thing I did: **one script per format, not one script that grows a branch for every format.** It's less elegant on paper. It's dramatically easier to test, debug, and extend in practice — a Word-extraction bug can't touch the Excel path, or the original text path, if they were never sharing code to begin with.

## What Actually Runs

Four small, deliberately single-purpose scripts now — the original text-watching script, kept exactly as it was since it had never actually been the problem, plus three new ones for PDF, PowerPoint, and Word, added separately rather than folded into the same file. Excel came later still, as a fifth, once I noticed spreadsheets landing in Downloads and going unindexed the same way PDFs once had. Each script is responsible for exactly one input format, and each produces the same thing on the way out: a markdown file, written into the vault folder, ready for the next overnight pass to index the same way any other note would be.

The shape each one follows:

```python
# pdf_to_vault.py — one format, one job
import sys
from pathlib import Path
import pdfplumber

def extract(source_path: Path) -> str:
    with pdfplumber.open(source_path) as pdf:
        return "\n\n".join(page.extract_text() or "" for page in pdf.pages)

def write_note(source_path: Path, text: str, vault_dir: Path):
    note_path = vault_dir / f"{source_path.stem}.md"
    note_path.write_text(
        f"# {source_path.stem}\n\nSource: {source_path.name}\n\n{text}",
        encoding="utf-8"
    )

if __name__ == "__main__":
    source = Path(sys.argv[1])
    vault_dir = Path(r"D:\corporate_brain\vault")  # or your own vault path
    text = extract(source)
    write_note(source, text, vault_dir)
```

The other four follow the identical outer shape — read a file, extract text in whatever way actually suits that format, write a markdown note into the vault — with only the `extract()` function genuinely different between them. `pptx_to_vault.py` and `docx_to_vault.py` use `python-pptx` and `python-docx` respectively, each pulling text out of that format's own internal structure the way `pdfplumber` does for PDF above. `xlsx_to_vault.py` uses `openpyxl`, covered separately below since Excel's `extract()` function ended up doing something genuinely different from the other three. That's the whole point of splitting them apart: the boring, repeated plumbing (write a markdown file, name it sensibly, drop it in the vault folder) is identical across all five, and the only place format-specific complexity is allowed to live is inside that one function, quarantined from the other formats entirely.

I won't pretend every extraction function is trivial. Excel in particular needed its own judgment call. `xlsx_to_vault.py` uses `openpyxl` to open the workbook, but dumping a sheet as a wall of comma-separated values into a markdown note is technically "text extracted," and not genuinely useful to a model reading it later. A short structured summary — sheet names, column headers, row count, and a sample of the actual data — turned out to be far more useful than a full raw dump, the same way Chapter 5 found that model size alone doesn't tell you what a model's actually good for — raw completeness isn't the same thing as usefulness. Roughly:

```python
# xlsx_to_vault.py — Excel's extract() is a genuine exception, not a raw dump
import openpyxl

def extract(source_path: Path) -> str:
    wb = openpyxl.load_workbook(source_path, data_only=True)
    summary = []
    for sheet in wb.worksheets:
        headers = [cell.value for cell in sheet[1]]
        summary.append(
            f"## {sheet.title}\n"
            f"Rows: {sheet.max_row}, Columns: {sheet.max_column}\n"
            f"Headers: {headers}\n"
        )
    return "\n".join(summary)
```

## The Trigger: A Watched Downloads Folder

These scripts don't run on a schedule the way the overnight pipeline in Chapter 17 does. They're triggered by `watch-dropzone.ps1`, a PowerShell script kept running continuously via a Windows Task Scheduler entry (`SovereignBrain_Watcher`, launched through a small `launch-watcher.bat` wrapper) rather than firing on a timer — new file lands, its extension gets matched to the right one of the five scripts, that script runs against it, a markdown note appears in the vault. The overnight indexing pass from Chapter 17 doesn't need to know or care that the note originated from a PDF rather than being typed directly into Obsidian — by the time it sees the file, it's just another markdown note sitting in the vault folder like any other.

Worth being explicit about the wrapper, since it's a small but genuinely useful pattern: the scheduled task itself doesn't run PowerShell directly. It runs `launch-watcher.bat`, a one-line batch file whose only job is calling PowerShell with `-ExecutionPolicy Bypass -WindowStyle Hidden` and pointing it at the real script. That indirection exists because Windows Task Scheduler and PowerShell's own execution policy can be a genuine source of silent failures — a task that looks configured correctly but never actually runs its script because of a policy restriction that only shows up if you go looking for it. Wrapping the PowerShell call in a `.bat` file sidesteps that class of problem entirely, at the cost of one extra file to keep track of.

## The Tailscale Transfer Folder

Alongside Downloads, there's a second entry point into the same pipeline: a dedicated transfer folder on my desktop PC, fed over Tailscale file transfer from other devices — the same private network from Chapter 12, doing a second job beyond just remote HTTPS access to the stack. A file sent to that folder from my phone or another machine on the tailnet gets picked up the identical way a Downloads file does — same five scripts, same format-matching logic, same markdown-into-vault destination. I didn't build a second ingestion path for this; I pointed the same watcher at a second folder, which turned out to be the more useful decision than it might have seemed at the time, since it meant a fix to any one of the extraction scripts automatically covered both entry points rather than needing to be made twice.

## What "Done" Looks Like

Any PDF, PowerPoint, Word document, or Excel file — landing either in Downloads or in the Tailscale transfer folder — gets picked up automatically, converted into a markdown note, and made available to the vault's RAG integration by the next overnight pass, with no manual step in between. That's the gap this chapter closes: Chapter 15 explained what happens once something is a markdown note in the vault. This chapter explains how nearly everything else actually gets to that starting point.

Chapter 16 covers security guardrails — including the guardrail layer question that's been sitting open since n8n came out of this stack.
