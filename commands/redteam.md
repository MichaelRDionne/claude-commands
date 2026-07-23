---
description: Red-team the current request/task/artifact for missing assumptions, failure modes, and guardrails — then do the best lean version without overbuilding.
---

Apply this red-team check to the current request, immediately preceding task context, or the artifact supplied in the arguments below.

Arguments:

```text
$ARGUMENTS
```

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
