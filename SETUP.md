# fuller-smith.co.uk — Setup Guide

This is a scaffolded Hugo site: config, folder structure, and three article
stubs pre-loaded with front matter and your section outlines as comments.
Follow these steps in order.

## 1. Install Hugo (PowerShell)

```powershell
winget install Hugo.Hugo.Extended
hugo version   # confirm it installed
```

## 2. Turn this folder into a git repo and add the theme

From inside this `fuller-smith-site` folder:

```powershell
git init
git add .
git commit -m "Initial site scaffold"

# Add PaperMod as a submodule (clean, fast, minimal — good fit for
# text-heavy technical content, no ad-network styling to fight)
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod --depth=1
```

If you'd rather try Congo (also minimal, slightly more modern typography)
instead, swap the URL for `https://github.com/jpanther/congo.git`.

## 3. Preview locally

```powershell
hugo server -D
```

`-D` includes draft posts (everything here is marked `draft: true` until
you're ready to publish it). Open the local URL it prints — usually
`http://localhost:1313`.

## 4. Review and publish

The four articles are already fully drafted in `content/posts/`:

- `local-ai-hardware-guide.md` (Article 1 — hardware guide)
- `model-to-vram-sizing-guide.md` (Article 2 — VRAM sizing guide)
- `lemonade-vs-llamacpp-vs-ollama.md` (Article 3 — inference engines)
- `agent-harnesses-openclaw-vs-hermes.md` (Article 4 — agent harnesses)

Each is marked `draft: true` in its front matter, so none of them go live until you flip that to `draft: false`. Read through a post in the local preview, make any edits you want directly in the file, then flip its draft flag when you're ready to publish it. To start a brand new post from scratch:

```powershell
hugo new content/posts/my-new-post.md
```

When a post is ready, flip `draft: true` to `draft: false` in its front
matter — Hugo won't build/deploy draft posts to production.

## 5. Push to GitHub

Create a new repo (private or public, your call) on GitHub, then:

```powershell
git remote add origin https://github.com/<your-username>/fuller-smith-site.git
git branch -M main
git push -u origin main
```

## 6. Connect Cloudflare Pages

1. Cloudflare dashboard → Workers & Pages → Create → Pages → Connect to Git
2. Select the `fuller-smith-site` repo
3. Build settings:
   - Framework preset: **Hugo**
   - Build command: `hugo --gc --minify`
   - Output directory: `public`
   - Environment variable: `HUGO_VERSION` = (whatever `hugo version` printed)
4. Deploy — Cloudflare builds and gives you a `*.pages.dev` URL to confirm
   it works before touching DNS.

## 7. Point the domain at it

If `fuller-smith.co.uk` isn't already on Cloudflare DNS:

1. Cloudflare dashboard → Add a site → enter `fuller-smith.co.uk`
2. Cloudflare gives you two nameservers — update these at your current
   registrar (wherever you bought the domain)
3. DNS propagation can take a few hours

Once the domain is on Cloudflare:

1. Workers & Pages → your project → Custom domains → Add `www.fuller-smith.co.uk`
   (and `fuller-smith.co.uk` if you want the bare root to work too)
2. Cloudflare handles the SSL certificate automatically

## 8. Ongoing workflow

```powershell
# write/edit a post, then:
git add .
git commit -m "Draft: hardware guide intro"
git push
# Cloudflare Pages auto-builds and deploys on every push to main
```

## Notes

- Everything here costs £0/month beyond the domain you already own.
- Draft posts stay invisible until you flip `draft: false` — safe to
  commit and push work-in-progress.
- `enableGitInfo = true` in `hugo.toml` means Hugo can show "last
  updated" dates on posts if the theme supports it — useful given the
  hardware/pricing sections are flagged for periodic refresh.
