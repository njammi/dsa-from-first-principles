# Decisions

Settled, long-term decisions for this project. Once something is here, treat it as
fixed — don't re-debate it in future sessions unless we explicitly revisit it.
Format: newest at the top.

---

## D-004 · Publish with GitHub Pages via GitHub Actions
**Date:** 2026-07-04
The site auto-deploys on every push to `main` through `.github/workflows/deploy.yml`
(build with `mkdocs build --strict`, publish with `actions/deploy-pages`). Pages is
configured with build type = "GitHub Actions" (not the legacy branch build).
**Why:** zero manual publishing; a bad build fails CI instead of shipping broken pages.
**Note:** GitHub Pages caches for 10 min (`max-age=600`) — hard-refresh or use `?v=N`
to see updates immediately; the `mkdocs serve` tunnel is the instant preview.

## D-003 · Repository is public, named `dsa-from-first-principles`
**Date:** 2026-07-04
Public repo under GitHub user `njammi`. Live at
https://njammi.github.io/dsa-from-first-principles/
**Why:** the book is meant to be published; the repo doubles as the GitHub learning
resource named in the mission. Public is also required for free GitHub Pages.

## D-002 · Site engine is Material for MkDocs
**Date:** 2026-07-04
Articles are Markdown in `docs/`, built by MkDocs with the Material theme.
**Why:** Python-native (fits the Python-only project), excellent code highlighting +
copy buttons, built-in search, light/dark, sidebar nav, admonitions — ideal for a
code-heavy technical book. Config lives in `mkdocs.yml`.
**Consequence:** article files live under `docs/part-N-.../`; nav is maintained in
`mkdocs.yml` and must be updated when adding an article.

## D-001 · Foundational project rules (from CLAUDE.md + PERSONA.md)
**Date:** 2026-07-02
Python 3 only; strict WHY → WHAT → HOW → CODE → OPTIMIZATION order; first-principles
teaching; the fixed 20-section article structure; the repository is the source of
truth, not chat history. See CLAUDE.md and PERSONA.md for the full statements.
