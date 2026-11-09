---
description: Red-team the current request/task/artifact for missing assumptions, failure modes, and guardrails — then do the best lean version without overbuilding.
---

Apply this red-team check to the current request, immediately preceding task context, or the artifact supplied in the arguments below.

Arguments:

```text
$ARGUMENTS
```

## The lean check — always run this

Before answering or acting, identify what the user may be missing:

- risky assumptions
- hidden failure modes
- missing inputs
- needed guardrails or approval gates
- privacy, sensitive-data, credential, browser, file, repo, external-system, or destructive-action risk
- validation or test checks needed
- anything that should become a script, template, skill, slash command, or reusable workflow

Then proceed with the best lean version of the task. Preserve the user's intent, do not overbuild, and make the smallest useful improvement.

If ambiguity is low-risk, make a reasonable assumption and state it. If the action is destructive, privacy-sensitive, external-facing, or could touch the wrong files/accounts/systems, stop and ask first.

## The two-seat roster — when the artifact is worth a real adversary

A single model red-teaming its own work grades its own homework. Two seats with different jobs catch different classes of defect:

| Seat | Good at | Owns |
|---|---|---|
| **Scanner** — a fast, long-context model | breadth, whole-repo and whole-manuscript continuity, syntax and regex verification, speed | finding every place the problem occurs |
| **Judge** — your strongest reasoning tier | state machines, rule precedence, boundary conditions, invariants | deciding which findings are real, and writing the fix |

The scanner owns coverage and the judge owns adjudication. Handing both jobs to one model collapses the check, because the seat that surfaced a candidate is the worst seat to ask whether it matters.

**Announce the roster before you dispatch**, so the reader knows which seats actually ran:

```markdown
[RED TEAM DISPATCH: TWO-SEAT CHECK]
• Target: <artifact name, file path, or task topic>
• Seats: <scanner model> (Scanner) + <judge model> (Judge)
• Transport: <how each seat was reached>
• Protocol: (1) Risky Assumptions, (2) Hidden Invariant Violations, (3) Concrete Drop-In Fix
```

Label the banner with the seats that answered. A banner naming a seat that quietly failed over to something else is the one error nothing downstream can catch, because every later reader takes the attribution at face value.

## Output contract

Every red-team output reports exactly three sections:

1. **Risky assumptions (max 3)** — fragile or unverified premises.
2. **Hidden invariant violations and failure modes** — concrete edge cases, rule-precedence collisions, state desync, and statutory or trademark exposure.
3. **Concrete drop-in fix** — the exact replacement code, configuration, or text. A theoretical critique with no working replacement does not count as a finding.

Capping assumptions at three is deliberate. An unbounded list is how a red team launders low-confidence guesses into the same register as the one real defect.

## On ensemble size

An earlier version of this fanned every check out to four or five separate model CLIs. That mode is retired, and two seats carry the standard check now.

One lane stays available on demand. For bulk sweeps where priming is the real risk — grading a long manuscript or an exam bank against a key — send an unprimed model the material blind, with no access to the key and no access to the other seats' findings. A blind seat cannot be led, which is the whole reason to spend a third dispatch on one.
