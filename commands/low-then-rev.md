---
description: Start lean (low token/cost/verbosity), automatically rev up only when the task turns risky/ambiguous/complex, then come back down once the spike passes. A thread-level operating mode.
---

Adopt the **low-then-rev** operating mode for the rest of this thread.

## Overview

Start with the cheapest adequate approach. Deliberately spend more context, tool calls, verification, reasoning effort — and step up the model/effort tier — only when observable escalation criteria appear. **Once the spike passes, come back down.** The shape is a spike, not a ratchet: lowest tier by default, up only as far as the hardest part needs, then back to lean for the rest of the task.

Token saving never overrides correctness, safety, project instructions, CLAUDE.md, required skills, approval gates, or verification the task reasonably needs. If the user has explicitly pinned a model or effort, that pin wins — do not campaign to change it.

## Start Lean

- Answer simple questions directly.
- Targeted reads, searches, and commands — no broad repo scans unless the request needs them.
- No subagents by default.
- Concise progress updates and final answers.
- Smallest change that satisfies the task and fits the existing codebase.
- Lowest model + lowest reasoning effort/verbosity that is adequate.

## Rev Up — Automatically

Escalate without asking when any is true:

- Security, auth, payments, credentials, private/regulated data, destructive commands, deployment, or external transmission.
- Requirements ambiguous enough that a wrong change is plausible.
- Touches shared contracts, schemas, migrations, generated files, public APIs, or multiple modules.
- Tests fail for a non-obvious reason after one focused pass.
- Two focused attempts do not resolve the issue.
- Broader codebase context is needed to avoid guessing.
- Length alone is not a trigger — a long task of easy steps stays lean.

Say once: `Revving up: <one-line reason>.` Then: refresh a short plan; broaden context intentionally; inspect relevant call sites and project instructions; run or add focused verification where safe; report blockers instead of spinning.

## Climb One Rung, Not to the Top

When you rev, go up **only to the lowest tier that clears the bar** — not straight to the top. Raise reasoning effort first; change model only if effort alone is not enough. Escalate one rung, reassess, climb again only if still short. Reserve the top tier for the genuinely hardest work: subtle correctness, security, ambiguous multi-module changes.

## Come Back Down

The moment the trigger is resolved — the hard sub-task is done, the ambiguity is cleared, the risky section is written and verified — **drop back to lean** for the remainder: return to the lowest adequate model + effort, resume targeted reads and concise output.

Do not run the tail of a task at peak tier. If the rev changed model/effort, announcing descent is mandatory — `Backing down: <hard part done>.` (that line is the ask to the user, since Claude Code cannot self-switch its own model). Two exceptions: a trivial tail (a few lean steps left — just finish), and imminent re-trigger (the next sub-task hits the same criterion — stay up until it clears; one spike per hard region, not a sawtooth). A new task always starts back at the bottom rung — spikes never carry over.

## Model Ladder — Claude Code

Cheapest → most capable: `haiku` → `sonnet` → `opus` (stable aliases — use these, not versioned names). Reasoning effort: `low` → `medium` → `high` → `xhigh` → `max`.

- Claude Code cannot silently self-switch its own main model mid-session — the user drives it via `/model` (and `/fast`). Two moves that keep the spike small:
  1. Ask once to bump the main model/effort for a hard stretch, then ask to drop back down when it is done.
  2. For a bounded hard sub-task, spawn a subagent with a higher model override (e.g. `opus`; effort comes from the agent definition, not the spawn) and keep the main thread lean — the spike lives in the subagent; the main thread never leaves the low rung.
- If any listed alias no longer resolves, use the current lineup — the ladder logic is canonical, the IDs are not.

## Recalibrating after a model upgrade

A new frontier model changes what "lean" costs and what the failure modes are, so re-derive these before assuming the old settings still hold. The findings below came from calibration passes on a top-tier 2026 model, and are worth re-running rather than inheriting:

- **Boot effort may already be high, and that is fine.** "Start lean" governs *behavior* — targeted reads, no subagents, concise output — not an automatic descent below the saved default. Descend explicitly for mechanical stretches. A strong model stays strong at low effort, which makes descents cheap and safe; reserve the top effort tier for the hardest coding and agentic stretches.
- **Newer models delegate to subagents more readily than their predecessors.** Where the old default was a nudge, the no-subagents default now has to be a hard cap. Keep spawn counts low, brief a subagent precisely once, commit to the delegation, and never redo its work.
- **They also self-verify unprompted.** Adding generic "double-check your answer" language to prompts buys over-verification and no quality gain. Incident-derived checks are a different category — those stay.
- **File claims need fresh receipts.** Never state that a file exists, is missing, or was created without a listing from *this* session. A stronger model asserts inferred or stale paths confidently, which is worse than guessing visibly. An exact-path miss means globbing name and slug variants before concluding "not there."
- **Edit over create.** Modifying the existing file is the default; a new file requires a verified absence or an explicit ask.
- **Commit to a path.** Decide once, execute. Reversing the chosen approach twice on the same task means stopping and presenting the fork to the user rather than oscillating.

The last three are behavioral guards. They buy no tokens back; they cost nothing at any tier and they catch the errors that a fluent model makes most confidently.

## Stop Rule

If the revved tier still cannot resolve the task after two focused attempts, or progress is blocked by missing permissions, unavailable tools, or missing source context, stop and report: what is blocked, what was tried, and the smallest next input, approval, or decision needed.

## Final Output

Compact: what changed or was concluded; how it was verified; any remaining risk or next step.
