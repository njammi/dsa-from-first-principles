# CLAUDE.md — DSA from First Principles

This repository follows the **Claude Project Operating System (CPOS) v2**.

Canonical standard: `../AI-Workspace-Architect/STANDARDS.md` — all behavioral rules (startup, scope discipline, token efficiency, decisions, confidence, end-of-session) are defined there and apply here in full.

Everything below is **project-specific only**.

---

## Mission

A public book teaching Data Structures & Algorithms from first principles, written as MkDocs articles and published to GitHub Pages at https://njammi.github.io/dsa-from-first-principles/. Quality bar: every article follows the 20-section structure in PERSONA.md.

## Startup

After the CPOS startup procedure (PERSONA.md, then WORKSPACE.md), WORKSPACE.md tells you the current status and which files to read next. No other files are needed at startup.

## Project Rules

- WORKSPACE.md is the operational source of truth: where things live, the preview/publish commands, and the add-a-new-article checklist. Follow that checklist exactly when adding articles.
- Standing FAQ rule: reader questions about article content get appended to `docs/faq.md` (details in WORKSPACE.md).
- Verify before publishing: `mkdocs build --strict` must pass, and run any code samples in the article.

## Do Not

- Never edit anything under `site/` — it is generated build output (git-ignored; CI rebuilds it).
- Never push to `main` without being asked — pushing deploys the public site.
