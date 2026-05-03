# Scaffolding a fresh Zensical notes site

When the destination folder is empty (or has only a `.git`/`README.md`), use these templates to bootstrap. They produce a working site that runs immediately with `zensical serve` (or `docker compose up`) and follows the conventions the rest of this skill assumes.

Replace anything in `<angle-brackets>` with the real value before writing the file.

## `zensical.toml`

```toml
[project]
site_name = "<Book Title> — Reading Notes"
site_description = "Detailed reading notes on <Book Title> by <Authors> (<Publisher>, <Year>)."
site_author = "<User name or handle>"
copyright = "Copyright © <Year> <User>. Notes summarize concepts from <Book Title> by <Authors> (<Publisher>, <Year>)."

# Hierarchical navigation. Each entry maps a display title to either a markdown
# file (relative to docs/) or a nested array for sub-sections.
nav = [
    { "Home" = "index.md" },
    { "Chapters" = [
        { "1. <Chapter 1 title>" = "chapters/01-<slug>.md" },
        { "2. <Chapter 2 title>" = "chapters/02-<slug>.md" },
        # … one per chapter
    ]},
    { "Reference" = [
        { "Glossary"  = "reference/glossary.md" },
        { "Resources" = "reference/resources.md" },
    ]},
]

# NOTE: [project.theme] is intentionally omitted. Zensical 0.0.x ships only its
# default theme by name; setting name = "modern" or "classic" causes a
# "Theme '<x>' is not installed" error in current releases. Once Zensical 0.1+
# ships, you can add e.g.:
#
#   [project.theme]
#   name = "modern"
#   language = "en"
```

For *very long* books, consider grouping chapters into "Parts" using nested arrays — but only if the book itself is structured that way. Don't manufacture parts that aren't in the source.

```toml
{ "Part I — Getting Started" = [
    { "1. <…>" = "chapters/01-<slug>.md" },
    { "2. <…>" = "chapters/02-<slug>.md" },
]},
```

## `docs/index.md`

```markdown
# <Book Title> — Reading Notes

Detailed reading notes for **<Book Title>** by <Authors> (<Publisher>, <Year>). Written as a **detailed outline** — each chapter captures the structure, key concepts, and the specific commands or syntax worth remembering.

## How these notes are organized

<1–2 sentences describing how the chapters group, if relevant. Otherwise drop this section.>

## Reading log

| Status | Chapter |
| ------ | ------- |
| ⬜ todo   | 1. <Chapter 1 title> |
| ⬜ todo   | 2. <Chapter 2 title> |
| ⬜ todo   | 3. <Chapter 3 title> |
| …      | … |

## Conventions

- **Concepts** are introduced as short, declarative bullets.
- **Commands / syntax** appear in fenced code blocks.
- **Open questions / things to revisit** are marked with `> ❓`.
- **Version-adapted content** is flagged with a 📌 callout at the top of the chapter.

## About the source book

> <Authors> (<Year>). *<Full title with subtitle>* (<edition>). <Publisher>. ISBN <…>.
```

The reading-log table is the single most useful affordance on the home page. As chapters get done, flip `⬜ todo` to `✅ done`. Don't remove rows; you want the full chapter list visible.

## `docs/chapters/<NN>-<slug>.md` (placeholder)

For every chapter the user *hasn't* studied yet, create a placeholder. This keeps the navigation working and gives the reading log somewhere to point.

```markdown
# Chapter <N> — <Chapter Title>

> *Source: <Authors> (<Year>), Chapter <N>.*
>
> Notes pending — this chapter hasn't been studied yet.

## Outline placeholder

- [ ] Read the chapter
- [ ] Capture key concepts
- [ ] Note new commands / syntax
- [ ] Record open questions

## References

_(to be added)_
```

## `docs/reference/glossary.md`

Start empty (or seeded if you have one chapter done). Add a section per chapter as terms are introduced.

```markdown
# Glossary

A running glossary of terms as they appear across the book. Updated chapter by chapter.

## From Chapter 1

| Term | Meaning |
| --- | --- |
| <Term> | <Short, standalone definition.> |
```

Tip: keep definitions standalone — i.e. someone reading just the glossary entry should understand the concept without having to flip to the chapter. That's what makes a glossary useful.

## `docs/reference/resources.md`

```markdown
# Resources

Links collected from the book — a single place to find them later.

## Official <Topic> docs

- <Title> — <https://…>

## Tooling

- <Tool name> — <https://…>

## The book

- Source code — <https://…>
- Errata / updates — <https://…>
```

## `README.md` (project root)

```markdown
# <Book Title> — Reading Notes

A [Zensical](https://zensical.org/) static site built from reading notes on *<Book Title>* by <Authors> (<Publisher>, <Year>).

## Run with Docker (recommended)

```bash
docker compose up
# open http://localhost:8000
```

`zensical.toml` and `docs/` are bind-mounted, so edits live-reload.

## Run locally with Python

```bash
python -m venv .venv
. .venv/Scripts/Activate.ps1   # macOS/Linux: source .venv/bin/activate
pip install zensical
zensical serve
```

## Adding a new chapter's notes

1. Edit the file under `docs/chapters/`.
2. The navigation is already wired up in `zensical.toml`.
3. Update the reading-log table in `docs/index.md`.
```

## `Dockerfile`

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Zensical is pre-1.0 (0.0.x); pin under 0.1 to avoid surprises if a 0.1
# breaking release lands.
RUN pip install --no-cache-dir "zensical>=0.0.30,<0.1"

EXPOSE 8000
CMD ["zensical", "serve", "--dev-addr", "0.0.0.0:8000"]
```

## `docker-compose.yml`

```yaml
services:
  docs:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: <project-slug>-docs
    ports:
      - "8000:8000"
    volumes:
      - ./zensical.toml:/app/zensical.toml
      - ./docs:/app/docs
    restart: unless-stopped
```

## `.dockerignore`

```
.venv/
__pycache__/
*.pyc
site/
.zensical/
.git/
.vscode/
.idea/
.DS_Store
Thumbs.db
node_modules/
```

## `.gitignore`

```
site/
.zensical/
.venv/
__pycache__/
*.pyc
.vscode/
.idea/
.DS_Store
Thumbs.db
```

## After scaffolding

Tell the user the site is ready and how to run it. Then proceed straight into Chapter 1 (or whichever chapter they asked for) using `note-style.md` as the guide.

Don't create more than one project per book by default. If they're studying a second book, ask whether they want a sibling folder (e.g., `~/Notes/postgres/`, `~/Notes/kubernetes/`) or a single bigger site that hosts both — both are reasonable but have different navigation implications.
