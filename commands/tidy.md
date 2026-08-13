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
3. **Known-positive gate before the word "clean" gets drafted.** Plant a synthetic fixture in the workspace that exercises **every** pattern class you are about to scan for — one line per class. Confirm each class flags it. Then delete the fixture and verify the deletion. If any class misses, the sweep is broken; fix it before scanning for real. Never write "clean" for a class that has not demonstrably gone red once this run, and note that a truncated display (`head -N`) is not a class result — confirm against unbounded output or a count.

Rule 3 was added after the third recurrence of the same shape. Rules 1 and 2 were each written in response to a sweep that reported green and was wrong, and each time the fix was to widen the patterns. Widening does not prove the widened check runs. Only a fixture that goes red does.

Also open the **cache payloads** — plugin caches, tool logs, page-snapshot directories. Deleting a source file does not remove its cache record; in the originating setup a sensitive name survived in a plugin cache after every other copy was gone. And report the **copy surfaces**: sync services, backups, and version history a finding has already reached — "it's only local" is usually false.

**A plain grep or read on a PDF or other binary file returns garbage or nothing — that is not a content check.** Extract the text layer first (`pdftotext <file> -` or an equivalent) before treating a PDF as scanned. That extraction has its own blind spot: it reads the text layer only, so a fillable or prefilled form's field *values* are invisible to it — in the originating setup a prefilled application produced over a thousand words of extracted text and zero hits on a field that was, in fact, filled in more than twenty places. For fillable or prefilled PDFs, also dump the form-field values with a forms-aware library, and report field names and fill status rather than the values themselves. Where no extractor ran at all, say so explicitly — "binary payloads: existence confirmed, content not checked" — rather than letting a directory listing imply text-level coverage it never had.

Report paths and identifier *categories* with counts. Never quote the sensitive content itself. Findings are Tier-B: disposition is the operator's call, and any evacuation must be checksum-verified at the destination before anything is deleted.

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
5. **Orphans & duplicates (Tier B).** Files nothing links to, and obvious copies (`foo (1).md`, `foo-copy.md`). Many orphans are legitimate — present each with a one-line "what it is" and let the operator pick keepers.

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

Without `--apply`: stop here. With `--apply`: show the Tier-A list, ask "delete these N files? (y/n)", and only on an explicit yes, `rm` exactly those files — one by one, echoing each. Never expand the delete set beyond what the report listed.

## Rules

- **Report before prune, always.**
- **Tier B is never auto-deleted.** Orphans, duplicates, links, and every sensitive-data finding are judgment calls for the operator.
- **Prefer reversible.** Everything Tier-A should be regenerable (build artifacts, editor backups, OS cruft). If a "stray" file looks like real content, demote it to Tier-B and ask.
- **Don't refactor.** This is housekeeping — no moving or renaming content files.
- If you later schedule it, schedule the **report only**. `--apply` never runs unattended.
