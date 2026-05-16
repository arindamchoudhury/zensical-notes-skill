# zensical-notes — Claude Code skill

A [Claude Code](https://claude.ai/code) skill for turning PDF books into a personal study site built with [Zensical](https://zensical.org/).

You upload a PDF (a technical book, paper, manual), say *"let's do chapter 3"*, and the skill produces structured Markdown notes, keeps the site navigation in sync, maintains a running glossary and resources list, and adapts any version-specific content to the current release of the technology.

## What it does

- **Detailed-outline notes** — hierarchical, navigable, written in your own words. Not a summary, not a quote dump — something you'll actually come back to.
- **Multi-book hybrid layout** — per-book chapter notes plus cross-book topic pages. Start with one book; add a second and the structure is already ready.
- **Version-aware** — detects version-bound content (install commands, "what's new" sections, EOL dates) and updates it to the current release via a web search at note-taking time.
- **Chapter page map** — on first use, extracts all chapter boundaries from the PDF in one pass and saves them to `page-map.md`. Every future session reads the map instead of re-scanning the PDF.
- **Chapter text cache** — extracted chapter text is saved to `pdf-cache/` on first extraction; subsequent sessions read from the cache and skip `pdftotext` entirely.
- **Research cache** — web search results (version lookups, API facts) are saved to `docs/research-cache/` with a "Last verified" date so future sessions skip redundant searches.
- **Math support** — configures `pymdownx.arithmatex` + MathJax so `$...$` and `$$...$$` render in the browser. Scaffold templates include the JS config file and `extra_javascript` wiring.
- **Site sync** — after each chapter, automatically updates the reading log, glossary, resources list, and topic index.
- **Hot reload on Docker Desktop for Windows** — includes a `livereload`-based `serve.py` workaround for the inotify/WSL2 limitation that prevents `zensical serve` from detecting file changes on Windows.

## Requirements

- [Claude Code](https://claude.ai/code) (CLI or IDE extension)
- [`pdftotext`](https://poppler.freedesktop.org/) on the machine Claude Code runs on (`poppler-utils` on Debian/Ubuntu, `brew install poppler` on macOS)
- [Zensical](https://zensical.org/) (`pip install zensical`) — or Docker (the scaffold templates include a ready-to-use `Dockerfile` and `docker-compose.yml`)

## Installation

### Option A — download and install as a `.skill` file

1. Download this repository as a ZIP: **Code → Download ZIP**
2. Rename the downloaded file from `zensical-notes-skill-main.zip` to `zensical-notes.skill`
3. Place it somewhere permanent, e.g. `~/.claude/skills/zensical-notes.skill`
4. Register it in your Claude Code user settings (`~/.claude/settings.json`):

```json
{
  "skills": [
    { "path": "~/.claude/skills/zensical-notes.skill" }
  ]
}
```

### Option B — clone and point at the directory

Clone the repo and point Claude Code at the directory directly in `settings.json`:

```json
{
  "skills": [
    { "path": "/path/to/zensical-notes-skill" }
  ]
}
```

## Usage

Once installed, trigger the skill by describing what you want:

```
"Let's take notes on this PDF — chapter 1 first."
"I'm studying Kubernetes, let's do chapter 4."
"Add this book to my notes site."
"Next chapter."
```

Claude Code will invoke the skill automatically. You can also invoke it explicitly:

```
/zensical-notes
```

## Project layout produced

```
<notes-root>/
├── zensical.toml
├── docs/
│   ├── index.md               # hub page
│   ├── topics/
│   │   ├── index.md           # topic backlog → full pages once ≥2 books cover them
│   │   └── <topic>.md
│   ├── books/
│   │   └── <slug>/
│   │       ├── index.md       # reading log
│   │       ├── page-map.md    # PDF page ranges (extracted once, reused every session)
│   │       ├── pdf-cache/     # raw extracted chapter text (reused across sessions)
│   │       │   ├── 01-<slug>.txt
│   │       │   └── …
│   │       └── chapters/
│   │           ├── 01-<slug>.md
│   │           └── …
│   ├── reference/
│   │   ├── glossary.md
│   │   └── resources.md
│   └── research-cache/        # verified version/API facts (not wired into nav)
├── docs/javascripts/
│   └── mathjax.js             # MathJax config (only present if notes contain equations)
├── serve.py                   # livereload-based dev server for Docker on Windows
├── Dockerfile
├── docker-compose.yml
└── README.md
```

## Reference files

The skill includes four reference documents read on demand:

| File | Purpose |
| --- | --- |
| `references/note-style.md` | Full style guide for chapter outlines, with an annotated example |
| `references/scaffold.md` | Drop-in templates for `zensical.toml`, index pages, glossary, `serve.py`, Dockerfile, etc. |
| `references/version-handling.md` | Rules for adapting version-specific book content to the current release |
| `references/multi-book.md` | Multi-book hybrid layout: migration steps, topic-page workflow, cross-linking |

## License

MIT — see [LICENSE](LICENSE).
