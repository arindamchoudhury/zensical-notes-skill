---
name: zensical-notes
description: Build and extend a Zensical "reading notes" site from PDF books, papers, or technical manuals — including multi-book setups studying one topic across several sources. Trigger whenever the user wants to take notes on a PDF, study chapter-by-chapter, summarize a book into durable docs, extend an existing notes site, add a second book to an existing site, or compare what multiple books say about a topic. Catches phrases like "take notes on this PDF", "let's do chapter X", "summarize this chapter", "I'm studying [topic]", "add this book to my notes", "compare these books on X", and any time a user uploads a PDF and asks for note-taking help — even without naming Zensical. Produces detailed-outline Markdown, scaffolds single- or multi-book projects (per-book chapters plus cross-book topic pages), keeps nav/reading-log/glossary/topic-pages in sync, and refreshes version-specific content to the latest release at note-taking time.
---

# Zensical Notes

A workflow for turning PDFs into a personal study site built with [Zensical](https://zensical.org/) — the static-site generator from the Material for MkDocs team. The output is a folder of plain Markdown files plus a `zensical.toml` config; the user runs the site locally with `zensical serve` (or via Docker) and gets a clean, searchable, navigable docs site of their notes.

## When to use this skill

Use it whenever the user wants to *capture* knowledge from a PDF in a way they'll come back to. Strong cues:

- They've uploaded a PDF (a book, paper, manual, RFC, course handout) and want to study it.
- They mention chapter-by-chapter work: "let's do chapter 2", "next chapter", "give me a detailed outline".
- They mention an existing notes folder, or a Zensical / MkDocs project.
- They're studying a technology and want notes that stay useful as the technology evolves.
- They want to study the same topic across multiple books and have one place to think across them.

This skill is not for: short Q&A on a PDF, extracting a single quote, translating, OCR, or filling forms — defer to the **pdf** skill for those. If the user wants a Word doc or PowerPoint instead, defer to **docx** / **pptx**.

## Project layout

There are two supported layouts. Detect which one applies before doing significant work — `ls` of the notes root usually answers it.

### Single-book layout (default)

Use when one site = one book.

```
<notes-root>/
├── zensical.toml         # site name, theme, hierarchical nav
├── docs/
│   ├── index.md          # landing page + reading log table
│   ├── chapters/
│   │   ├── 01-<slug>.md  # one file per chapter (the meat of the notes)
│   │   ├── 02-<slug>.md
│   │   └── …
│   └── reference/
│       ├── glossary.md   # running glossary of terms across chapters
│       └── resources.md  # links collected from the book
├── Dockerfile            # optional — `zensical serve` in a container
├── docker-compose.yml    # optional — bind-mounts docs/ for live reload
└── README.md             # how to run the site
```

**Why this layout works:** chapter notes live under one predictable folder so the navigation in `zensical.toml` is easy to maintain; cross-chapter knowledge (terminology, links) lives separately under `reference/` so it doesn't get buried inside any one chapter; the site config is a single TOML file.

See `references/scaffold.md` for the full templates to use when scaffolding a fresh single-book site.

### Multi-book hybrid layout (per-book + topic pages)

Use when the user is studying the **same topic from multiple books** and wants both per-book chapter notes (faithful to each author's narrative) *and* cross-book topic pages (synthesized perspective).

```
<notes-root>/
├── zensical.toml
├── docs/
│   ├── index.md           # hub linking to both Topics and Books
│   ├── topics/
│   │   ├── index.md       # topic list + "topics to write" backlog
│   │   ├── mvcc.md        # ← Ferrari Ch 11 + Rogov Ch 4
│   │   └── …
│   ├── books/
│   │   ├── ferrari/
│   │   │   ├── index.md      # per-book reading log
│   │   │   ├── page-map.md   # PDF page ranges (one-time extraction, reused every session)
│   │   │   ├── pdf-cache/    # raw extracted chapter text (reused across sessions)
│   │   │   │   ├── 01-<slug>.txt
│   │   │   │   └── …
│   │   │   └── chapters/01-…md
│   │   └── rogov/
│   │       ├── index.md
│   │       └── chapters/04-mvcc.md
│   └── reference/
│       ├── glossary.md    # merged across books, attributed
│       └── resources.md
└── …
```

**Detection:** if `docs/books/` exists with at least one sub-folder, you're in multi-book mode — read `references/multi-book.md` immediately and follow its workflow rather than trying to scaffold a single-book site on top.

**Trigger to migrate single → multi:** when a single-book project exists and the user asks to add a *second* book. See the migration section of `references/multi-book.md` — do the migration *before* starting on the new book, and verify the site still builds.

## The flavors of work

Almost every invocation of this skill is one of these:

1. **First time with a book, fresh site** — no notes folder yet. Scaffold a *single-book* project (templates in `references/scaffold.md`), read the PDF TOC, lay out a chapter file per chapter as a placeholder, write detailed notes for the requested chapter, point them at the running site.

2. **Adding a chapter to an existing single-book site** — `zensical.toml` and `docs/chapters/` already exist. You're filling in one more chapter file, updating the reading-log row in `docs/index.md`, and possibly adding terms to the glossary / links to resources.

3. **Adding a *new book* to an existing single-book site** — the user is now studying a second source. **Migrate** the site to the multi-book hybrid layout *before* taking notes on the new book. See the migration section of `references/multi-book.md`.

4. **Adding a chapter inside a multi-book site** — `docs/books/<slug>/` exists. Read `references/multi-book.md` if you haven't this session. Write the per-book chapter notes, then do the topic-page pass: identify which topics the chapter touches, decide whether to extend an existing topic page, create a new one (only when ≥2 books overlap), or add to the topic backlog. Cross-link bidirectionally.

Always confirm which flavor applies before doing significant work. A quick `ls` of the notes root usually answers it (`docs/books/` present → multi-book; `docs/chapters/` present → single-book; nothing → fresh).

## The detailed-outline note style

This is the only style this skill produces by default. It's a hierarchical outline that captures the *structure* of the chapter — sections, subsections, the few things in each subsection that actually matter — written in prose-y bullets and short paragraphs rather than a flat summary or quote dump.

Why this style and not "just summarize"? A reader returns to study notes mostly to *find a thing* — a definition, a command, the subtle distinction between two concepts. A faithful structural outline mirrors the chapter's own table of contents, so locating that thing later takes seconds. Pure prose summaries collapse the structure and force re-reading.

Why this style and not "extract every example"? Because cluttering the page with verbatim long passages defeats the point of *notes* — and risks copyright trouble on commercial books. Code blocks should be kept short; longer passages get *paraphrased* in your own words, with a page reference if helpful.

Read `references/note-style.md` for the full style guide, including:

- The exact section ordering for a chapter file (preface block → numbered sections → summary → references).
- How to render code, callouts, sidebars, and version notes.
- How to mark "open questions" and "things to revisit".
- A complete annotated example chapter you can model from.

Read it the first time you write a chapter in a session, then keep going.

## Working with the PDF

PDF extraction is what the **pdf** skill is for — read its SKILL.md if you need detail. The minimum you need here:

- For a chapter's contents, `pdftotext -layout` produces clean enough text for note-taking. You don't need OCR unless the PDF is scanned.
- Don't paste the raw text back to the user. Read it, internalize it, and produce the outline.

If the user uploaded the file, it lives under the conversation's uploads folder. If they're pointing at a file in the connected workspace, use that path directly.

### Chapter page map — extract once, reuse every session

When starting a new book project, build a chapter boundary table in one pass and save it to `docs/books/<slug>/page-map.md`. Every subsequent session reads that file instead of re-running `pdftotext` to find page ranges.

**Step 1 — find the PDF-to-print-page offset.** Printed page numbers almost always differ from PDF page numbers (front matter shifts everything). Scan the first ~30 pages to find a known landmark:

```bash
pdftotext -f 1 -l 30 -layout "<book>.pdf" -
```

If PDF page 34 shows "1" as the printed page number, the offset is +33 (PDF page = printed page + 33).

**Step 2 — enumerate chapter start pages.** Pipe the full text through `grep` to spot chapter headings:

```bash
pdftotext -layout "<book>.pdf" - | grep -n "^Chapter " | head -30
```

The line numbers correspond to *output text lines*, not PDF pages — use them to triangulate, then confirm by extracting a small range with `-f`/`-l`.

**Step 3 — save to `page-map.md`.**

```markdown
# Page map — <Book Title>

PDF-to-print offset: +33  (PDF page 34 = printed page 1)

| Chapter | Title | PDF start | PDF end | Printed pages |
| --- | --- | --- | --- | --- |
| 1 | Introduction | 34 | 53 | 1–20 |
| 2 | Getting Started | 54 | 86 | 21–53 |
| … | … | … | … | … |
```

In subsequent sessions: read `page-map.md`, look up the target chapter's PDF start/end, extract with `pdftotext -f <start> -l <end> -layout "<book>.pdf" -`.

> **Don't extract all chapter text upfront.** The full PDF text is far too large for context. Extract one chapter at a time, immediately before writing its notes. The page map is cheap (a small table); bulk pre-extraction is wasteful and unnecessary.

### Chapter text cache — extract once, read many times

Once a chapter's text is extracted, save it so it never needs re-extracting in a future session.

**Before extracting,** check whether `docs/books/<slug>/pdf-cache/<NN>-<slug>.txt` already exists. If it does, read it directly — skip `pdftotext` entirely.

**If the cache file is missing,** extract and save in one step:

```bash
pdftotext -f <start> -l <end> -layout "<book>.pdf" \
  "docs/books/<slug>/pdf-cache/<NN>-<slug>.txt"
```

Then read from the saved file.

**Naming:** use the same `<NN>-<slug>` stem as the chapter `.md` file — `01-introduction.txt`, `05-advanced-statements.txt`. The pairing is then obvious at a glance.

> 💡 The cache contains raw book text. Consider adding `docs/books/*/pdf-cache/` to `.gitignore` if you don't want to commit extracted text to version control.

## Version-aware notes (the key trick)

Books — especially technical books — go stale. The user is studying *now*; the book might cover PostgreSQL 16, Python 3.11, Kubernetes 1.28, while the current release is one or two majors newer. The point of these notes is to be useful *now and going forward*, so any version-specific content needs to be aligned with reality at note-taking time.

The behavior:

1. **Detect version-bound content.** As you read a chapter, watch for: explicit version numbers ("PostgreSQL 16"), release-date phrases ("released October 2023"), "what's new in vN" sections, install commands that pin a version (`postgresql-16`, `python3.11`), EOL/lifecycle claims, and version-comparison sidebars.

2. **Confirm what's current with a web search.** Use the `WebSearch` (or `web_fetch`) tool to look up the *current stable release* of the relevant technology. One search per technology is usually enough — search official sources first (`docs.python.org`, `postgresql.org`, project's own site) before secondary blogs.

3. **Adapt — don't rewrite the chapter.** Keep the chapter's structure and concepts (those rarely change). Update only the version-bound parts:
   - Replace the version number where the book uses "the current version".
   - Replace the "what's new in vN" section with a "what's new in v<latest>" section, sourced from the project's release notes.
   - Update install commands to the current major's package names / repo paths.
   - Update EOL math (release year + project's support window).
   - Update minor URLs that include version paths.

4. **Tell the reader what you did.** Add a note callout near the top of the chapter, e.g.:

   > 📌 **Notes adapted to <Tech> v<latest>.** The book covers v<book-version> (released <date>); v<latest> is the current production release as of <today>. Conceptual content is unchanged because the authors note that core concepts apply across versions.

   This is honest about the gap between the book and the notes, and gives a future-you reader a fighting chance of reconciling examples that don't quite match the book.

5. **Don't invent features.** If a search doesn't surface a clear answer for a specific feature, leave the book's content as-is and add a `> ❓ Revisit:` note. Hallucinating "what's new" content is worse than being honest about uncertainty.

The full decision rules and a worked example (PostgreSQL 16 → 18 adaptation) live in `references/version-handling.md`.

## Keeping the site in sync

A chapter note isn't done until the other site files are updated:

- **`zensical.toml`** — confirm the chapter is wired into `nav`. For new books this is the whole nav table; for new chapters the slot is usually already there. The file is TOML, not YAML — see `references/scaffold.md` for nested-nav syntax (single-book) and `references/multi-book.md` for the multi-book version.
- **Reading log** — flip the row from `⬜ todo` → `✅ done`. In single-book mode this lives in `docs/index.md`; in multi-book mode it's in `docs/books/<slug>/index.md`.
- **`docs/reference/glossary.md`** — append any *new* terms the chapter introduced. Don't rewrite existing entries unless they're now wrong. In multi-book mode, attribute each entry to its source book(s); when two books define a term differently, keep both.
- **`docs/reference/resources.md`** — append links the chapter cites that look useful long-term (project docs, repos, RFCs). Skip purely internal cross-references.
- **(multi-book only) Topic pages and topic index** — for each topic the chapter informs, either extend an existing topic page, create a new one (≥2 books overlap), or add to the backlog in `docs/topics/index.md`. Add the bidirectional cross-link between chapter note and topic page. See `references/multi-book.md`.

These updates are small, but skipping them is the #1 way the site rots.

## Naming and slugs

- **Chapter files:** `<NN>-<short-kebab-slug>.md` where `NN` is two-digit (`01`…`19`). Two digits because lexical sort matches numeric order up to 99 chapters. The slug should be 3–5 words from the chapter title.
- **Book slugs (multi-book mode):** short, kebab-case, ideally one word — `ferrari`, `momjian`, `rogov`. Author surname unless ambiguous.
- **Topic files (multi-book mode):** short noun phrase, no number — `mvcc.md`, `replication.md`, `query-tuning.md`. Topic order is editorial.
- **Display titles in `nav`** get the full chapter or topic title, not the slug. The slug is for the filesystem; the title is for humans.

## Hot reload on Docker Desktop for Windows

`zensical serve` uses inotify to detect file changes. Docker Desktop on Windows (WSL2 backend) does **not** propagate inotify events for bind-mounted volumes from the Windows host, so the server never detects edits — even though the files are correctly mounted. Manual browser reload also shows stale content.

**Fix:** replace `zensical serve` with a `serve.py` script that uses the `livereload` Python package, which polls file modification times (no inotify required) and injects a WebSocket snippet so the browser auto-refreshes.

`serve.py` (project root, COPY'd into the container):

```python
import subprocess
from livereload import Server


def build():
    subprocess.run(["zensical", "build"], check=True)


build()  # initial build on startup

server = Server()
server.watch("docs/", build)
server.watch("zensical.toml", build)
server.serve(root="site", port=8000, host="0.0.0.0")
```

`Dockerfile` changes — add `livereload` to the pip install line, COPY the script, and update CMD:

```dockerfile
RUN pip install --no-cache-dir "zensical>=0.0.30,<0.1" livereload
COPY serve.py /app/serve.py
CMD ["python", "serve.py"]
```

After adding these files, rebuild with `docker compose up --build`. The site now auto-refreshes in the browser on every save; no manual reload needed.

## Theme caveat (Zensical 0.0.x)

Zensical is pre-1.0 (0.0.x as of writing). Two practical consequences:

- **Don't set `[project.theme]` with `name = "modern"` or `"classic"`** unless the user has confirmed those themes are installed in their environment — the bundled-by-default themes referenced in newer docs aren't always present at install time. Leaving the `[project.theme]` block out makes Zensical use whatever it has built in, which is the safest default. See `references/scaffold.md` for the recommended config.
- **CLI flags can drift.** If a user reports "`zensical serve` doesn't accept `--dev-addr`," it's almost certainly a flag rename — guide them to `zensical serve --help` first.

Once Zensical hits 0.1+ this section can be revisited.

## A complete worked example

The user uploads a 745-page PDF of a Linux kernel internals book and says *"help me take notes on this, let's start with chapter 1."*

A good run looks like:

1. Confirm style/format and book layout if not obvious (one quick clarifying question to the user is fine — don't ask four).
2. Inspect the destination folder. If it's empty, scaffold the project (`zensical.toml`, `docs/index.md`, `docs/chapters/`, `docs/reference/`).
3. **Build the chapter page map** — extract chapter boundaries in one pass and save to `docs/books/<slug>/page-map.md` (see "Chapter page map" in "Working with the PDF"). This is a one-time cost per book; future sessions read the file directly.
4. Create placeholder chapter files for chapters 2..N so the navigation works from day one.
5. **Check the chapter text cache** — look for `docs/books/<slug>/pdf-cache/01-<slug>.txt`. If absent, extract and save: `pdftotext -f <start> -l <end> -layout "<book>.pdf" docs/books/<slug>/pdf-cache/01-<slug>.txt`. Then read from the file.
6. Detect version-bound content. Web-search for any current versions discussed.
7. Write the chapter outline using the style in `references/note-style.md`. Insert the version-adaptation note callout if any updates were made.
8. Update `zensical.toml` nav (if needed), reading log, `docs/reference/glossary.md` (new terms), `docs/reference/resources.md` (links).
9. Tell the user how to run the site (`docker compose up` or `zensical serve` — README in the project explains both) and ask whether to continue with chapter 2 next.

If the project is already in multi-book mode, step 2 is "read `references/multi-book.md`," step 7 also writes/extends a topic page when ≥2 books overlap, and step 8 also updates the topic index and adds the bidirectional cross-links.

## Anti-patterns to avoid

- **Don't re-run pdftotext if the cache exists.** Check `docs/books/<slug>/pdf-cache/<NN>-<slug>.txt` before extracting. A cache hit skips the extraction entirely and keeps the context window smaller.
- **Don't dump raw extracted text.** It's not notes; it's a copy. Re-state ideas in your own words.
- **Don't write a flat summary** of long chapters — keep the structure visible.
- **Don't rewrite chapters from scratch** when the book is just slightly out of date. Adapt the version-bound bits, leave concepts alone.
- **Don't quietly skip the reading-log update.** Future-you will lose track of what's done.
- **Don't pin versions unless asked.** A user studying technology generally wants notes against *current*, not whatever happened to be latest the day they ran the skill — but that's a soft preference. If they ask for "as of when the book was written," respect it.
- **Don't pretend to know latest versions from training.** Always web-search when version-currency matters.
- **(multi-book) Don't write topic pages with only one source.** Keep them in the backlog until a second book provides the comparison point — otherwise the topic page is just a chapter summary in disguise.

## Reference files

Read these on demand. Each one stands alone.

- `references/note-style.md` — full style guide for chapter outlines, with an annotated example. Read once per session at minimum.
- `references/scaffold.md` — drop-in templates for `zensical.toml`, `docs/index.md`, glossary, resources, README, and Dockerfile/compose. Read when scaffolding a new single-book project.
- `references/version-handling.md` — decision rules for adapting a book's version-specific content to current reality, with a worked PostgreSQL 16 → 18 example. Read when the chapter mentions any version, release, or install command.
- `references/multi-book.md` — the hybrid per-book + topic-page layout: when to use it, how to migrate from single-book, the topic-page template, the cross-linking convention, and glossary handling across books. Read whenever `docs/books/` exists, or whenever the user is adding a second book to an existing site.
