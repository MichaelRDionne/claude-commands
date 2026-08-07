---
description: Generate a paste-ready prompt for an in-browser AI assistant to run a task on a page behind your logged-in session — for when a task needs an authenticated web surface (a portal, an LMS, webmail) and a fresh automated-browser tab would just hit a login wall.
---

# /sidecar — generate an authenticated-session relay prompt

Most browser-automation tools drive a *new* browser context — which means a *new* login. A page that needs your real session (SSO, a paywall, a portal that doesn't offer API access) stops them cold. The workaround: you already have that session open in an ordinary browser tab, alongside an in-browser AI assistant that can act inside it. This command writes the prompt that assistant runs, so the terminal agent never has to touch your credentials at all.

**Core loop:** the terminal agent holds the task context and writes a precise, self-contained prompt → you paste it into the in-browser assistant running inside your authenticated tab → it executes on the live page and reports back as structured text → you paste the report back → the terminal agent reconciles it into wherever the result belongs.

In the originating setup this replaced a browser-automation tool that kept re-hitting the same login wall on a multi-folder scrape a scripted session simply couldn't reach.

## When to invoke

- Any browser task that needs your authenticated session: an LMS, a payer or benefits portal, webmail, a bank or credentialing site, anything behind SSO.
- A login wall is hit mid-task by whatever automated-browser tool you'd normally reach for.
- You say "use the sidecar" / "have the sidecar do it."

Do **not** use this for regulated or sensitive data (health records, anything under its own compliance pipeline) — that stays on its own gated path, never routed through a general-purpose relay like this one.

## Steps

### 1. Pick the tool, state the pick in one line

| # | Situation | Tool |
|---|---|---|
| 1 | Needs **your logged-in session** (SSO / portal / paywall) | **Sidecar relay** — only surface that carries your session |
| 2 | Public page, no login, read-only | A fetch tool, or a fresh automated-browser tab |
| 3 | Public page, multi-step interaction (forms, pagination) | A fresh automated-browser tab — built for driving; it isn't your session anyway |
| 4 | "What's on my screen right now?" one-shot | A screenshot tool (read-only verify — never plan multi-step from a screenshot) |
| 5 | Authenticated **and mutating** (submit/upload/edit/pay) | **Sidecar relay, split in two**: read/stage prompt first → you review → separate confirm-to-act prompt |

Tie-breaker: unsure if a page needs auth? Try the plain fetch once; on a login wall go straight to the sidecar rather than burning attempts on tools 2–3.

If the task doesn't need your session at all, say so and use the plain tool instead of generating a relay prompt.

### 2. Build the 7-block prompt

Every sidecar prompt contains these, in order, inside copy markers:

```
ROLE: You are running inside my authenticated browser session on <SITE>.
SCOPE: Operate ONLY on <domain + specific pages>. Do not navigate off-domain.
MODE: READ-ONLY. Do not submit, upload, edit, delete, or change account state.
  You MAY: click to expand/open, scroll, open folders/links in scope, trigger DOWNLOADS.
TASK: <numbered concrete steps — what to open, what to capture>
CAPTURE RULES: Quote instructions/dates/titles VERBATIM — no paraphrase. If text is
  truncated or a page won't load, say so explicitly; do not reconstruct.
REPORT FORMAT: Return one fenced block matching this schema:
  <explicit structure — e.g. per item: name / due date / verbatim text / attachments>
  End with: DOWNLOADS DISPATCHED: <list|none> / ISSUES: <list|none>.
RESET: When done, return the page to its start state (close panels, navigate back to
  <start URL>). Do not log out.
```

Default **MODE: READ-ONLY** unless a mutating action was explicitly requested (see below). The **report-format block is the highest-leverage one** — over-specify it so the paste-back reconciles in one pass with no follow-up round. Pre-answer predictable snags in the TASK block ("if a folder is empty, report `EMPTY` and continue"; "accept the default download filename") so the assistant never stalls asking you mid-run.

### 3. Firewall-scan before emitting

Scan the drafted prompt against your own banned-content categories before it leaves the terminal agent — anything from a compliance/legal firewall, any credential, any target that's actually a regulated-data surface. If it points at one, stop and redirect to that data's own gated pipeline instead of emitting a relay prompt.

### 4. Emit in copy markers

Wrap the prompt in clearly marked copy boundaries (no other formatting inside them). Below the block, print the **expected report schema** so you know what "done" looks like before you paste the prompt in.

### 5. On paste-back: reconcile

When the report comes back, reconcile it into wherever the result belongs (a tracker, a file, a downstream step). If downloads were dispatched, stage the **single follow-up action** — reveal the download location + the exact expected filenames as a checklist. The sidecar and the terminal agent both lack general filesystem access to your downloads, so that handoff has to happen by hand.

## Mutating actions (submit / upload / edit / pay)

A mutating relay is a **separate artifact**, generated only when explicitly requested:

- Exactly ONE mutating action per prompt.
- First line names it: "This prompt WILL submit/upload X."
- The assistant must describe what it's about to change and get your in-session confirmation before acting.
- Never combine capture and mutation in one prompt. Read/stage first → you review → confirm-to-act second.

## Known walls (do not fight these)

- **The sidecar has no general filesystem access.** It can dispatch downloads but never verify, rename, or move them — that's a human-in-Finder (or equivalent) step, unless you explicitly grant the terminal agent access to a specific dropped file.
- **The browser's own sandbox blocks the terminal from your downloads folder.** The agent can't list or move files there directly.
- **A screenshot is eyes-only.** Use it to confirm state, never to drive multi-step action.
- **The sidecar is async and unwakeable.** It only runs when you paste a prompt in; the terminal agent can't poll or trigger it. The report is the *only* record of a run — that's why verbatim capture and an `ISSUES:` footer are mandatory. The report has to be self-certifying, because nothing else confirms what happened.

## Design note

Keep it **stateless** — each invocation is one round-trip. No session tracking; the paste-back message is the entire state handoff. Target 2 pastes and at most 1 follow-up action: one prompt per round (chain every phase inside a single prompt rather than sending several), a machine-parseable report schema, and pre-answered snags are what keep it there.

## The transferable pattern

The core move: when a task needs a real authenticated session that automation can't cheaply replicate, don't fight for the session — write a precise, bounded, self-certifying prompt for whatever already has it, and treat the round-trip as the entire protocol. Read/mutate stays a hard split, the report schema is over-specified on purpose, and every emitted prompt gets scanned against your own sensitive-content rules before it leaves your control.
