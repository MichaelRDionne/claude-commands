---
description: Root mechanics on any model; the judge is a subagent for plan and verdict, and the judge's verdict is final.
---

# /captain — plan, dispatch, and verdict

The root runs on whatever model the session is on. The strongest tier is spawned as a subagent only where its judgment decides something: the plan and the verdict.

There is no refusal state. A non-judge root is the normal case, and it routes to the planner-subagent path.

An earlier version required the strongest model in the chair and refused otherwise — wrong twice over: it burned the top tier on mechanical turns and made a model-picker state a hard blocker.

**Never add a `model:` key to this file.** Frontmatter pins are per-turn overrides that the interactive interface does not apply; see `anthropics/claude-code#81318`.
The Agent tool's model override is a different mechanism with a working record.

## Step 0 — census as routing, not a gate

Run one census. Its only job is to choose the planning path:

```bash
jq -r 'select(.type=="assistant" and (.isSidechain!=true)) | .message.model' ~/.claude/projects/<project>/<session-id>.jsonl | sort | uniq -c
```

| Census result | Planning path |
|---|---|
| `<judge-model-id>` present | Plan **inline**; a planner subagent would be redundant. |
| No judge model | Use the **planner-subagent path**. |
| Empty, or the read fails | Use the **planner-subagent path**; inconclusive takes the safe branch at the cost of one subagent. |

State the result in one clause before the plan: “No judge model in root → spawning one judge planner subagent,” or “Judge model in root → planning inline, no planner subagent.”

Use YOUR OWN session id; a newest-file heuristic can select a sibling session's transcript.
Permission attachments record what was requested, never what ran.

**Proving a judge subagent actually served.** Subagent transcripts are separate `agent-<id>.jsonl` files, not sidechain entries in the parent. Census the agent file:

```bash
jq -r 'select(.type=="assistant") | .message.model' ~/.claude/projects/<project>/<session-id>/subagents/agent-<agent-id>.jsonl | sort | uniq -c
```

The sibling `agent-<id>.meta.json` `model` field records the request, not proof.
The `.message.model` census is the proof.

## The chair

**The judge's verdict is final and the root never overrides it**, whether it arrives inline or through a subagent.
If the root disagrees, relay the verdict verbatim, give the operator the contradicting evidence as information, and stop. Reopeners go through a fresh `/captain`.

The root is mechanical: it reads, runs, dispatches, and relays. It does not rule.

## Mandatory escalation — triggered, not judged

Escalation is triggered, not judged. The failure mode is a cheap root deciding it has a judgment call covered — documented in the originating setup.

These always go to the judge:

- Anything touching compliance rules, regulated data, disclosure-sensitive content, or your canonical rule set.
- Ambiguity resolution: two readings of the mission lead to different work.
- Cross-task synthesis: combining worker outputs into a conclusion.
- The final verdict: every mission, no exceptions.
- Any point where the root would override, revise, or set aside a judge output.

Everything else runs on the root. “I think I've got this” is never a reason to skip a trigger.

## Phase 1 — PLAN

**Inline path:** when the census shows the judge, work the problem with the operator directly.

**Planner-subagent path:** otherwise, gather current state cheaply, then spawn one judge planner subagent with a self-contained brief.

Include the mission, constraints, gathered context, tier table, lane rule, scarcity line, and a request for bounded tasks with a tier, lane, and one-word reason for each scarce-pool task.
The root relays the plan; it does not write it.

Either path produces bounded, independently verifiable tasks:

| Tier | Use for |
|---|---|
| `haiku` | Mechanical/bulk work: renames, format conversions, sweeps, boilerplate, simple extraction. |
| `sonnet` | Standard implementation, focused research, and drafting to a clear spec. |
| `opus` | Hard-but-bounded reasoning: multi-file refactors, tricky debugging, or one-component design. |
| your judge model (subagent) | The planner and the judge; also any mandatory-escalation trigger. |

If you also run a peer agent CLI on a separate subscription, add its cheap/default tiers to this table.
Route produce-lines-against-a-spec work there. Regulated data never leaves the lane carrying your compliance coverage.
State which pool is scarce so the planner prefers the other on close calls, and require its one-word tag so the vendor-mix check reduces to a table lookup.

**Force structure is the plan's call, including ZERO troops** when the work can run inline.
The mandatory judge does not turn a zero-troop mission into a troop mission.

**Vendor-mix check:** before presenting the plan, cross-check every scarce-pool task against its tag.
Use only `skill`, `connector`, `coverage`, or `judgment`; flag a missing or unsupported tag rather than silently re-tiering the plan.

**Announce the roster.** Before the approval gate, state the planner already spent, every planned worker tier, the judge at close, and the vendor tally.
The operator sees the full mix before work begins.

**Print the plan as visible assistant text, not only thinking.** Your strongest model's thinking may not persist where the operator can read it.

Ask for the go via **AskUserQuestion**:

1. Activate (Recommended)
2. Revise the plan
3. Abort
4. Let's chat about it

Spawn nothing before the operator picks Activate.

## Phase 2 — ACTIVATE

**Agent lane:** use the planned Agent tool override. Launch independent tasks in parallel.
Every prompt is self-contained: goal, constraints, relevant paths, definition of done, and “return raw findings/diffs; do not editorialize.” Restate the compliance constraints in every prompt; never assume a subagent reads project instructions.

**Regulated-data work:** paste your data-handling contract verbatim. Paraphrased masking clauses leaked repeatedly in the originating setup.
Constrain return shapes to counts and indices, never record-level prose.

**Peer-CLI lane — contract**

```text
cat > <scratch>/<label>.brief <<'BRIEF'
<self-contained execution brief; quotes and $ stay literal>
BRIEF
```

- Send that brief on stdin; never pass it as an argument, because quotes and `$` break there.
- Point the tool at one output file. Its answer exists only there; stdout may be empty and its transcript may be on stderr.
- Default the sandbox to read-only. Apply single-file outputs yourself.
- Never point the CLI at your regulated-data store.

## Phase 3 — VERDICT

The root first performs the mechanical review against the plan's definition of done. “Subagent says done” is a status claim, not proof.
Spot-check claims against the files. A peer-CLI output file is the tool's self-report: verify it against the files it touched, never its summary.

A failed or weak result gets ONE re-spawn at its tier with sharpened instructions. A second failure escalates one tier or comes inline.

**Then the judge gives the verdict — mandatory for every mission.** Inline when the census shows the judge; otherwise spawn one judge subagent with the plan, execution record, mechanical-review findings, and any conflicts.

Relay the verdict lines verbatim in a marked block before any root commentary, then treat them as settled.

**Regulated-data missions:** de-identify the judge brief with cohort indices, counts, gate names and outcomes, non-record paths, and dispositions.
Carry the data-handling contract verbatim. **Exception:** if judgment cannot resolve without identifying material, the verdict stays inline with the root and the close-out discloses it.

## Close-out declarations

Record these repo-local declarations in your session log:

- `Deviations from the judge's plan: none` — or the list, including any spawn the approved roster did not name.
- If any mission turn ended without the Phase 1 approval call: `/captain deviation: Phase 1 ended without AskUserQuestion.` Name the reason if there was one.

Also record the planning path, the judge path, and the vendor tally actually dispatched.

## Token discipline

- Keep root context lean: gather, dispatch, relay. Put bulk reading and iteration loops in subagents.
- Judge-tier tokens cover the plan and the verdict; the mission's bulk work runs on cheaper tiers.
- At 30% context, stop opening phases or waves; finish in-flight work, give the verdict on what completed, and list what remains. The hard ceiling is 35%; split an oversized mission in Phase 1.
- This composes with `/low-then-rev`: it governs thinking effort; `/captain` governs who works and where judgment lands.
