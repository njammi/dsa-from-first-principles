# Data Structures & Algorithms, From First Principles

A beginner-friendly yet technically deep DSA book, written in Python and built to be read as one coherent book rather than disconnected tutorials. Every topic is explained from first principles: **why the problem exists → what the idea is → how it works → the code → the optimization.**

📖 **Read it online:** https://njammi.github.io/dsa-from-first-principles/

## Structure

- `docs/` — the articles (Markdown), organized by part.
- `mkdocs.yml` — site configuration (Material for MkDocs).
- `ROADMAP.md` — the learning sequence and what's planned next.
- The site auto-deploys to GitHub Pages on every push to `main` via GitHub Actions.

## Local preview

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve      # open http://localhost:8000
```

All code is Python 3. Start with Part 0 and read in order — each article builds on the last.
