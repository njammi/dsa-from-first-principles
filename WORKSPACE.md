# Workspace

**Read this first each session.** It's the "you are here" file: current status, where
things live, and the exact commands to preview and publish. For the fixed rules see
[CLAUDE.md](CLAUDE.md) and [PERSONA.md](PERSONA.md); for settled choices see
[DECISIONS.md](DECISIONS.md); for the full plan see [ROADMAP.md](ROADMAP.md).

---

## Status (updated 2026-07-04)

- **Live site:** https://njammi.github.io/dsa-from-first-principles/
- **Repo:** https://github.com/njammi/dsa-from-first-principles (public, `main`)
- **Published articles:**
  - Part 0 · 01 — How Computer Memory Works ✅
  - Part 0 · 02 — Dynamic Arrays ✅
  - Part 0 · 03 — Big-O Notation ✅
- **Next up:** Part 0 · 04 — Strings.

## Where things live

- `docs/` — the articles (Markdown), by part. Article files: `docs/part-N-name/NN-slug.md`.
- `mkdocs.yml` — site config **and the nav**. Adding an article = add the file *and* a nav entry here.
- `docs/index.md` — the landing page (also lists articles; update when adding one).
- Build output goes to `site/` (git-ignored; CI regenerates it).

## Preview locally (instant, always fresh)

MkDocs runs in a venv at the scratchpad path (no system Python pollution). To preview:

```bash
# from /home/ubuntu/dsa
<venv>/bin/python -m mkdocs serve -a 127.0.0.1:8000
```

Then from your laptop, tunnel in and open http://localhost:8000 :

```bash
ssh -L 8000:127.0.0.1:8000 <user>@<ec2-host>
```

The preview auto-reloads on save. (If the venv is gone: `python3 -m venv .venv` then
install `pip install -r requirements.txt` — note system Python needs
`--break-system-packages` or a venv due to PEP 668.)

## Publish

```bash
git add -A && git commit -m "..." && git push
```

Pushing `main` triggers GitHub Actions → builds → deploys to Pages (~1 min). The public
site caches for 10 min; hard-refresh or append `?v=N` to force-refresh.

## Adding a new article — checklist

1. Write `docs/part-N-.../NN-slug.md` (follow the 20-section structure in PERSONA.md).
2. Add its nav entry in `mkdocs.yml`.
3. Add it to the list in `docs/index.md`.
4. Update status in `ROADMAP.md` and this file.
5. Verify: `mkdocs build --strict` passes; run any code samples.
6. Commit + push.
