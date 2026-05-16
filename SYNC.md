# Keeping this repo in sync with the installed skill

The source of truth is the installed skill at:

```
$HOME\.claude\skills\zensical-notes\
```

This repo mirrors that directory. The files that must stay identical:

| Repo file | Skill file |
| --- | --- |
| `SKILL.md` | `SKILL.md` |
| `references/multi-book.md` | `references/multi-book.md` |
| `references/note-style.md` | `references/note-style.md` |
| `references/scaffold.md` | `references/scaffold.md` |
| `references/version-handling.md` | `references/version-handling.md` |

`README.md`, `LICENSE`, `.gitignore`, and `SYNC.md` are repo-only — they don't exist in the skill directory.

## Check for differences

```powershell
$skill = "$HOME\.claude\skills\zensical-notes"
$repo  = "$PSScriptRoot"

diff "$repo\SKILL.md"                        "$skill\SKILL.md"
diff "$repo\references\multi-book.md"        "$skill\references\multi-book.md"
diff "$repo\references\note-style.md"        "$skill\references\note-style.md"
diff "$repo\references\scaffold.md"          "$skill\references\scaffold.md"
diff "$repo\references\version-handling.md"  "$skill\references\version-handling.md"
```

## Pull from skill → repo

Run this after editing the skill files directly:

```powershell
$skill = "$HOME\.claude\skills\zensical-notes"
$repo  = "$PSScriptRoot"

Copy-Item "$skill\SKILL.md"                        "$repo\SKILL.md"
Copy-Item "$skill\references\multi-book.md"        "$repo\references\multi-book.md"
Copy-Item "$skill\references\note-style.md"        "$repo\references\note-style.md"
Copy-Item "$skill\references\scaffold.md"          "$repo\references\scaffold.md"
Copy-Item "$skill\references\version-handling.md"  "$repo\references\version-handling.md"
```

## Push from repo → skill

Run this after editing in the repo (e.g. after a pull request merge):

```powershell
$skill = "$HOME\.claude\skills\zensical-notes"
$repo  = "$PSScriptRoot"

Copy-Item "$repo\SKILL.md"                        "$skill\SKILL.md"
Copy-Item "$repo\references\multi-book.md"        "$skill\references\multi-book.md"
Copy-Item "$repo\references\note-style.md"        "$skill\references\note-style.md"
Copy-Item "$repo\references\scaffold.md"          "$skill\references\scaffold.md"
Copy-Item "$repo\references\version-handling.md"  "$skill\references\version-handling.md"
```
