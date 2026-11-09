---
description: Workspace housekeeping — surface stray files, backup cruft, orphans, duplicates, and broken links as a review list. Report-only by default; --apply prunes only the safe tier after an explicit yes. Never bundled into a session close-out.
---

# /tidy — workspace housekeeping (report-first)

Surface the clutter that accumulates in a long-lived working directory — stray root files, backup/scratch cruft, orphaned notes, duplicates, broken internal links — as a **review list**. Deletion is a deliberate, reversible, confirmed step, never an automatic side effect.

**This is standalone. It is NOT part of any session-handoff or close-out flow.** Cleanup at low-context end-of-session is how accidental deletions happen; keep it explicit and on-demand.

## Modes

- **`/tidy`** (default) — **report only.** Scans and produces the review list. Changes nothing.
- **`/tidy --apply`** — after showing the report and getting an explicit yes, prune **only the Tier-A safe list**. Tier-B items are surfaced but never auto-deleted, even with `--apply`.

## Hard exclusions — never scan for deletion, never touch

Adapt this list to your workspace, then treat it as non-negotiable:

- **Git internals** — `.git`, hooks, any separated git dir. Never prune or rewrite.
- **Agent-instruction and rule files** — `CLAUDE.md`, `AGENTS.md`, canonical rule documents, anything another agent treats as its entry point.
- **Permanent records** — append-only logs, immutable archives, templates. Flag oddities there; never prune them.
- **Regulated or sensitive data stores** — any volume or directory holding protected data is out of scope entirely. "Entirely" means **no access of any kind from this command, not even a read-only existence check** (`ls`, `test -f`) while tracing a link. Resolve links that point into such a store from your workspace's own documentation; if that can't settle it, report the link as unverifiable. A fix pass in the originating setup ran two `ls` probes against a protected mount while chasing dead links, which is how this line got written.
- Never delete a file this command didn't positively classify as Tier-A. When unsure → Tier-B (report only).

## Step 0 — sensitive-data sweep (run FIRST; data safety outranks tidiness)

Before touching clutter, sweep the workspace for sensitive content sitting where it shouldn't be. Three rules, all paid for with real incidents in the originating setup:

1. **No directory exclusions on detection.** You may exclude a directory from *deletion* (permanent record), never from *detection*. The worst finding in the originating setup sat in a directory the previous sweep had excluded as "obviously fine."
2. **Match every identifier format, not one.** A sweep that greps for one naming convention reports green while free-text variants of the same data sail past. *A check narrower than the rule it enforces reports green.* Search names AND initials, dates in every separator style (dashes, slashes, month names), phone numbers in every form, IDs, and both file contents and filenames.
3. **Known-positive gate before the word "clean" gets drafted.** Plant a synthetic fixture in the workspace that exercises **every** pattern class you are about to scan for — one line per class. Confirm each class flags it. Then delete the fixture and verify the deletion. If any class misses, the sweep is broken; fix it before scanning for real. Derive the fixture and the pattern set from the same class list, or the fixture plants a line no pattern was ever written to match and the miss reads as a pass. Near-identical formats count as separate classes: two spellings of the same unit need two patterns, each written to accept an unbounded number. Never write "clean" for a class that has not demonstrably gone red once this run, and note that a truncated display (`head -N`) is not a class result — confirm against unbounded output or a count. Delete the fixture before you compute any identifier census that goes into the report, or subtract its row explicitly. One report in the originating setup shipped a census table with the fixture still counted in it.

Rule 3 was added after the third recurrence of the same shape. Rules 1 and 2 were each written in response to a sweep that reported green and was wrong, and each time the fix was to widen the patterns. Widening does not prove the widened check runs. Only a fixture that goes red does.

Also open the **cache payloads** — plugin caches, tool logs, page-snapshot directories. Deleting a source file does not remove its cache record; in the originating setup a sensitive name survived in a plugin cache after every other copy was gone. And report the **copy surfaces**: sync services, backups, and version history a finding has already reached — "it's only local" is usually false.

**A plain grep or read on a PDF or other binary file returns garbage or nothing — that is not a content check.** Extract the text layer first (`pdftotext <file> -` or an equivalent) before treating a PDF as scanned. That extraction has its own blind spot: it reads the text layer only, so a fillable or prefilled form's field *values* are invisible to it — in the originating setup a prefilled application produced over a thousand words of extracted text and zero hits on a field that was, in fact, filled in more than twenty places. For fillable or prefilled PDFs, also dump the form-field values with a forms-aware library, and report field names and fill status rather than the values themselves. Where no extractor ran at all, say so explicitly — "binary payloads: existence confirmed, content not checked" — rather than letting a directory listing imply text-level coverage it never had.

Image-only PDFs that have already been OCR'd and content-checked can be recorded in a receipt table so later runs skip the re-OCR, on two conditions, or the table recreates the false-clean it replaces. **The hash is the key and the path is informational:** a file that moved but hashes the same is still checked, and a file at the same path with a new hash is not. **The hash is recomputed every run.** A run that trusts the table without re-hashing has asserted a stale receipt. OCR output goes to scratch only, and the source PDF is never rewritten; confirm that with a hash before and after.

Report paths and identifier *categories* with counts. Never quote the sensitive content itself. Findings are Tier-B: disposition is the operator's call, and any evacuation must be checksum-verified at the destination before anything is deleted.

## Before the first scan — locate your own copies

This command tends to exist in several places at once: one copy per agent runtime, each under its own directory convention. Before editing any of them or reporting on what exists, glob for every copy and wait for the result.

**Do not write a sentence that names a path until the tool call naming it has returned.** A same-turn `ls` that has not come back yet does not license the claim, and "the file is at X" is a claim whether it appears in a report or in conversation. Report `MISSING` for the copies that are absent and edit only the ones that are there.

**An exact-path miss in one runtime's directory is not a miss everywhere.** The failure this prevents is narrow and common: a lookup at one convention returns nothing, the run concludes the file does not exist, and a second copy that was never checked keeps running the old behavior indefinitely. Two copies of a housekeeping rule disagreeing about what is safe to delete is a worse state than either copy alone.

## The scan

1. **Stray root files (Tier A).** Define what the root of your workspace should contain; anything else loose there (`.zip`, scratch `.md`, exports, screenshots) is flagged "stray in root."
2. **Backup/temp cruft (Tier A).** `*.bak`-shaped files, `*.tmp`, `*.orig`, `*~`, `.DS_Store`, and empty non-index files:
   ```bash
   find . -type d \( -name .git -o -path './archives*' -o -path './protected-store*' \) -prune -o \
     -type f \( -iname '*bak*' -o -iname '*backup*' -o -name '*.tmp' -o -name '*.orig' \
     -o -name '*.old' -o -name '*.save' -o -name '*~' -o -name '.DS_Store' \) -print
   find . -type d \( -name .git -o -path './archives*' -o -path './protected-store*' \) -prune -o \
     -type f -empty ! -name 'README.md' ! -name 'INDEX.md' -print
   ```
   **Every scan command names the protected store in its own prune list** — including the throwaway second one that only looks for empty files. Replace `./protected-store*` with the real path of yours; a prune pattern matching nothing is the same as no prune at all. Step 0's sweep excludes it explicitly; a later `find` that doesn't inherits the exclusion only for as long as the operator re-derives it correctly, every run, from memory. In the originating setup that gap was live: the cruft scan walked into a mounted encrypted container and surfaced one internal filename before the operator caught it, corrected the command, and re-ran clean. No content was read and nothing was deleted. The fix still belongs in the command text, because a hard exclusion you have to remember is a soft one.

   The narrow form of this pattern (`-name '*.bak'` alone) missed a real backup file in the originating setup — a suffix like `something.doctor-bak` has no dot before `bak`, so an exact-suffix match sails past it. The broadened form above catches that shape, plus editor auto-backups (`Backup of *.docx`), at the cost of also matching legitimate files: backup *scripts*, directories named for what they back up. Triage the broadened matches by hand rather than bulk-classifying them Tier-A. And before calling any match a duplicate, checksum it against the file it looks like a copy of — an older revision is not a duplicate, and more than one "obvious backup" in the originating setup turned out to be draft history worth keeping.
3. **Untriaged inbox (route, don't prune).** If your workspace has a triage folder, report the count and route to whatever files it — filing is that flow's job, not this one's.
4. **Broken internal links (Tier B).** Spot-check high-traffic files for links that resolve to nothing; report file + line as fix candidates. Keep it bounded — don't lint every link every run.

   Four parser rules before you call a link broken. Each one was written after a proposed "fix" that would have corrupted files that were already correct, or after a skip that hid real links.

   1. Inside a Markdown table, the escaped pipe in `[[target\|alias]]` is required syntax. Strip the backslash to resolve, then leave the file alone. A checker that "repairs" it breaks the table. Watch your own capture regex here: one that stops at the pipe (`\[\[([^\]\|#]+)`) leaves the escape backslash at the end of the captured target, so a correct link reads as malformed. Strip the trailing backslash from the capture. Two runs in the originating setup reported these as "trailing-backslash-malformed links" and proposed editing three correct files. Describe the fix as un-escaping a capture, and grep the files for the literal `\]]` before believing it is on disk.
   2. A target carrying a tool-specific double extension (`foo.excalidraw`) lives on disk as `foo.excalidraw.md`. Check for the doubled form before flagging.
   3. If the target already ends in some other extension — `.txt`, `.sh`, `.pdf`, `.yaml`, `.json`, `.py` — resolve that path as written, relative paths included. Do not swap the extension for your default. In the originating setup a naive `Path.with_suffix(".md")` rewrote a real, existing `.txt` receipt into a phantom broken link, and the report read as a finding.
   4. A `[[` capture is not a link in exactly two shapes, so skip these and report nothing: the first character is whitespace (a shell `[[ … ]]` test, usually in a fenced code block), or the whole capture matches `^[\d.\s,+-]+$` and contains a comma (a JSON coordinate array). The comma is required. Without it the numeric rule also swallows date- and year-named notes like `[[2026-09-03]]` and `[[2027]]`. Match the shape, never the first character: the first draft of this rule in the originating setup skipped anything starting with a digit, `-`, or `.`, and measured before shipping it silently dropped 502 real captures, about 18% of the scanned surface, including digit-prefixed note names, every `../`-relative path, and rule 1's own test case. The shape match hits the same nine intended captures and nothing else. A first-character test is a checker that reports green by not looking.

   Then subtract the false-positive classes before you print a count: cross-workspace provenance pointers that intentionally leave the tree, template placeholders, and **the checker's own documentation**. A link checker that walks its own instructions reports the literal examples in them — `[[wikilink]]`, `[[target]]` — as broken, every run, forever. Where a link points into a protected store, report `exists=0/1` and nothing else; never print the target name.
5. **Orphans & duplicates (Tier B).** Files nothing links to, and obvious copies (`foo (1).md`, `foo-copy.md`). Many orphans are legitimate — present each with a one-line "what it is" and let the operator pick keepers.

   **An orphan count without its method is not comparable run to run.** Two honest passes over the same cache in the originating setup reported 158 orphans of 4,628 and 304 of 4,773. Both were correctly described as "the orphan count." Print which one you ran: whether composite keys are split before resolution, whether any large opaque prefix was excluded, and whether resolution is exact-path only or falls back to a basename glob. An unreported glob fallback is the dangerous one — it manufactures false "still backed by a real file" reads and shrinks the number for free.

   **Never run an existence probe across an opaque cache prefix.** A plugin cache in the originating setup held 1,033 keys under a single encrypted-container prefix. A naive orphan check calls `os.path.exists()` on every one of them, which is precisely the existence probe this command's own hard exclusions forbid — the key names are opaque and the bytes are ciphertext, so nothing leaks from the cache itself; the defect is the probe. Filter the prefix out before probing and report its count as one flat number.

## The report

```
## Tidy — [date]  (report-only | --apply)

### Tier A — safe to prune (reversible; deleted only on --apply + your yes)
- <path> — stray in root / backup cruft / empty file — [size, mtime]
  → N files, ~K total

### Tier B — review only (never auto-deleted)
- Sensitive-data findings: [category + count + copy surfaces reached] — operator's call
- Broken links: <file>:<line> → <missing target>
- Orphans: <path> — [one-line what-it-is]
- Duplicates: <a> vs <b> — [which looks canonical]
```

Without `--apply`: stop here. With `--apply`: show the Tier-A list, ask "delete these N files? (y/n)", and only on an explicit yes, `rm` exactly those files — one by one, echoing each. When N is 0, still write "delete these 0?" into the apply receipt; do not invent a yes and do not wait for one. Never expand the delete set beyond what the report listed.

## Rules

- **Report before prune, always.**
- **Tier B is never auto-deleted.** Orphans, duplicates, links, and every sensitive-data finding are judgment calls for the operator.
- **Prefer reversible.** Everything Tier-A should be regenerable (build artifacts, editor backups, OS cruft). If a "stray" file looks like real content, demote it to Tier-B and ask.
- **Don't refactor.** This is housekeeping — no moving or renaming content files.
- **Delete the sweep's own scratch at close-out.** Step 0's working files — per-class hit lists, full inventories, link dumps — are a derived concentration of exactly what the sweep was hunting, sitting in a directory with none of the protections the originals had. The run that produced them is the run that has to remove them. One pass in the originating setup honored the text-extraction scratch rule and still left 44 files and 1.9 MB of sweep scratch behind. `rm -rf` the run directory and confirm the deletion with an `ls` that misses.
- If you later schedule it, schedule the **report only**. `--apply` never runs unattended.
