---
description: Root mechanics on any model; the judge is a subagent for plan and verdict, and its verdict is strong advice the operator rules on.
---

# /captain — plan, dispatch, and verdict

The root runs on whatever model the session is on. The strongest tier is spawned as a subagent only where its judgment decides something: the plan and the verdict.

There is no refusal state. A non-judge root is the normal case, and it routes to the planner-subagent path.

An earlier version required the strongest model in the chair and refused otherwise — wrong twice over: it burned the top tier on mechanical turns and made a model-picker state a hard blocker.

**Never add a `model:` key to this file.** Frontmatter pins are per-turn overrides that the interactive interface does not apply; see `anthropics/claude-code#81318`.
The Agent tool's model override is a different mechanism with a working record.

If the argument block arrives empty, or reads back the literal placeholder token instead of expanding, the mission is the operator's most recent message. If that is not a mission either, ask for one and stop. An unexpanded placeholder is the tell that the command was invoked with no argument, and guessing a mission from surrounding context is how a session starts work nobody asked for.

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

The judge's verdict carries real weight without binding the root. It gets relayed before anything else, and the operator decides.

An earlier version of this file made the verdict final and barred the root from ever overriding it. That rule is retired. What replaced it is narrower and much harder to game: **the root never deviates silently.** If the root disagrees, it relays the verdict in the marked block first, then states the disagreement and the evidence behind it plainly, and the operator rules. Relaying a softened verdict, dropping an inconvenient line, acting on a disagreement before the operator has seen the original, and burying the disagreement in a handoff rather than saying it out loud are all the same failure wearing different clothes.

The root stays mechanical in the ways that matter: it reads, runs, dispatches, and relays. What it does not do is quietly substitute its own judgment and leave the operator reading a verdict that no longer describes what happened.

**Ground truth outranks any model's generated claim.** A judge running on a stale or hallucinated picture of the tree can assert a count, a file's existence, or a piece of syntax that a fresh check refutes on the spot. The root quotes the judge's line verbatim, states the command it ran and the output it got, applies what the disk says, and reports the discrepancy. The judge's word does not make a missing file exist.

That reaches facts and stops there. Acceptance criteria, design choices, step order, and a `FAIL` verdict are judgment. A passing test suite never overturns a `FAIL` — the judge may be failing the work on grounds the tests do not measure, which is most of why a judge is in the loop at all. And if applying a factual correction changes what work actually gets done, that is no longer a fact swap; stop and re-consult.

A root that already disagrees and then goes hunting for evidence has the order backwards. The disk check comes first, and the disagreement follows from what it returned.

## Mandatory escalation — triggered, not judged

Escalation is triggered, not judged. The failure mode is a cheap root deciding it has a judgment call covered — documented in the originating setup.

These always go to the judge:

- Anything touching compliance rules, regulated data, disclosure-sensitive content, or your canonical rule set.
- Ambiguity resolution: two readings of the mission lead to different work.
- Cross-task synthesis: combining worker outputs into a conclusion.
- The final verdict: every mission, no exceptions.
- Any point where the root would revise, skip, reorder, or set aside a plan step. Reordering feels like scheduling rather than judgment, which is exactly why it slips through.

**Anti-recursion invariant.** The plan must never invoke `/captain` or any circular orchestration, and subagents spawned during execution run at recursion depth zero. A planner that is allowed to schedule another planner produces a mission that looks like it is making progress while spending judge-tier tokens on coordination.

Everything else runs on the root. “I think I've got this” is never a reason to skip a trigger.

## Phase 1 — PLAN

**Inline path:** when the census shows the judge, work the problem with the operator directly.

**Planner-subagent path:** otherwise, gather current state cheaply, then spawn one judge planner subagent with a self-contained brief.

Include the mission, constraints, gathered context, tier table, lane rule, scarcity line, and a request for bounded tasks with a tier, lane, and one-word reason for each scarce-pool task.
The root relays the plan; it does not write it.

### Dispatching the judge

Try the Agent tool's model override first. If that call is rejected, never retry it with the model parameter stripped. A retry without the override silently plans on the wrong tier and looks like success. Fall back to the CLI wrapper instead.

- **Write the brief to a file and point the wrapper at the file.** Never pass a brief as a command-line argument; quotes and `$` do not survive the trip. Capture stderr to a sibling `.err` file, because some wrappers put their transcript there and leave stdout empty.
- **Cap the brief around 200 KB.** Past that, send `git diff --stat` for the whole change plus full hunks only for the files the plan actually named. An oversized brief fails in ways that read like a judgment.
- **The relayed block carries whichever judge actually answered.** If the wrapper failed over to a backup tier, the `.err` file says so, and the block has to carry the backup's name. Writing your first-choice judge's name on a block it did not produce is the one relay error that cannot be caught downstream — every later reader takes the attribution at face value.
- **A regulated-data gate returns its own exit code.** Stop and report it. Rewording the brief until it passes is defeating your own gate, and the reworded version is the one that leaks.
- **Any other failure, or empty output, is "judge unavailable" — report it and stop.** The root does not plan or rule in the judge's place. A missing judge is a blocked mission, and a mission the root quietly planned for itself is worse than one that stopped.

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

**Bare-maximal-instruction carve-out (operator ruling):** a bare maximal instruction from the operator ("fix all", "do all", "fix all remaining") licenses troop dispatch WITHOUT the plan-approval gate, provided the scope ruling is stated openly in the first reply and every named-risk act (external-system state changes, public pushes, deletions, new surfaces) still gets its own explicit approval question. Missions not opened with a maximal instruction keep the full gate.

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
- **A read-only sandbox flag is not containment. Treat every dispatch as write-capable.** Reproduced twice in the originating setup: a worker dispatched read-only edited three files in the live tree, and a minimal probe told to write a canary reported success with the canary sitting on disk afterward. Pass the flag — it costs nothing — but it may never again be the reason a dispatch is considered safe.
- **Containment has to be structural:** an isolated copy, a narrowed working directory, or a before/after manifest of the target tree. Prefer the manifest whenever the CLI runs against live paths, because it is the only one of the three that catches a write you did not predict. A receipt whose safety argument rested on the sandbox flag attests to the brief's declared scope and says nothing about what the process could reach.
- Single-file output survives as the safe pattern: the CLI writes to its output file and the root applies the edit. That safety comes from the root doing the write, which was never the sandbox's doing.
- **Read a CLI-produced test file in full before copying it into the real directory.** A test file is code that runs, and a skim is not a read.
- **Scratch files go to the session scratchpad, never `/tmp`.** Temp is reboot-volatile, so a fixture left there is gone before the next session can re-gate on it.
- Never point the CLI at your regulated-data store. Never disable approvals and sandboxing outright, and never pair a write-capable sandbox with a working directory at the root of your live tree.

## Phase 3 — VERDICT

The root first performs the mechanical review against the plan's definition of done. “Subagent says done” is a status claim, not proof.
Spot-check claims against the files. A peer-CLI output file is the tool's self-report: verify it against the files it touched, never its summary.

A failed or weak result gets ONE re-spawn at its tier with sharpened instructions. A second failure escalates one tier or comes inline.

**Fresh-maximal-instruction reopener (operator ruling):** a ruling the session minted FOR ITSELF (e.g. "no further root attempts — operator decides") is reopened by a fresh maximal instruction from the operator; that instruction sanctions ONE verbatim retry of each blocked item — one attempt, then stop — disclosed in the next verdict's deviations line. A maximal instruction is one of the named phrases ("fix all", "do all", "fix all remaining") or an explicit "retry the blocked items." A bare continuation word — "go", "ok", "proceed" — does not qualify, or every nudge reopens the whole blocked list.

**A permission-classifier denial is never reopenable this way.** A denial is the gate working. Only rulings the session minted for itself reopen here; a refusal handed down by the permission layer stands until the operator changes the permission, and a retry loop aimed at one is an agent routing around its own sandbox. Operator-decided calls stay closed here too. Those reopen only through the operator or a new mission.

**Then the judge gives the verdict — mandatory for every mission.** Inline when the census shows the judge; otherwise spawn one judge subagent with the plan, execution record, mechanical-review findings, and any conflicts.

Mandatory covers all four end states, not just the good one: success, partial, failed, and aborted. The abandoned mission is the one that most needs a verdict on the record, and it is the one sessions skip.

**What goes in the evidence section.** Inside a git repo, `git diff` plus test output. Outside one, the changed-file paths plus before/after excerpts of what changed. If the project has no tests, the brief says "no tests exist" in those words — an empty test section reads as a passing one.

**On `FAIL`, fix only the blockers the judge listed and resubmit once.** The adjacent thing you noticed while you were in there is a new mission. A second `FAIL` **on the verdict** ends the automated loop — hand the mission to the operator rather than starting a third pass, because two failed verdicts on the same work means the disagreement is not something another edit resolves. This is a different rule from the worker re-spawn above, which escalates a tier; a verdict has no higher tier to escalate to.

**Budget: one plan call and two verdict calls.** More than that needs the operator's go. The budget exists because a judge-tier loop is the most expensive way a mission can fail to converge, and it converges no better on the fifth call than the third.

Relay the verdict lines verbatim in a marked block before any root commentary. The block holds rulings, decisions, go/no-go calls, and any finding stated as a conclusion, quoted word for word and capped at 15 lines. Summarize the supporting reasoning in your own words after the block. Pasting the whole report spends context on analysis the operator did not need.

**Disagreement goes after the block, never inside it.** The block is the judge's words; your reading of them is yours. State the disagreement at full strength once, attach the evidence, and let the operator rule. Then execute what the operator decided — a second round of argument after a decision is filibustering.

**Re-run the Step 0 census immediately before you write the verdict.** The census taken at mission start records which model planned. A context compaction or a model switch partway through can land the verdict on a different model, and any close-out identity check reads the transcript rather than your memory of it. One `jq` line, right before the marked block.

**Regulated-data missions:** de-identify the judge brief with cohort indices, counts, gate names and outcomes, non-record paths, and dispositions.
Carry the data-handling contract verbatim. **Exception:** if judgment cannot resolve without identifying material, the verdict stays inline with the root and the close-out discloses it.

**A read-only brief does not cover mount state.** Add this clause verbatim to any brief that reaches a mounted volume: *"Do not attach, detach, mount, or unmount the volume. Work through the existing mount and leave mount state exactly as you found it."* In the originating setup a judge was briefed to write nothing, and it wrote nothing. It attached the encrypted container read-only to inspect it, then detached on exit, which unmounted the volume under the still-running root. The root's next write failed as a missing file on a path that had been there a minute earlier. "Read-only" constrains what a subagent puts into files. Mount state is not a file, so a subagent can honor a no-write brief to the letter and still pull the floor out from under you. Re-run your mount check after any volume-touching subagent returns, before your next write.

## Close-out declarations

Record these repo-local declarations in your session log:

- `Deviations from the judge's plan: none` — or the list, including any spawn the approved roster did not name, any factual correction ground truth triggered, and any point where the operator ruled against the verdict.

  **Write this as the last line of the verdict message itself.** Deferring it to the session handoff is how it goes missing: the handoff is written at low context, often by a different turn, and a deviation nobody wrote down at the moment it happened is a deviation nobody writes down at all. Name the judge that answered on the same line.
- If any mission turn ended without the Phase 1 approval call: `/captain deviation: Phase 1 ended without AskUserQuestion.` Name the reason if there was one.

Also record the planning path, the judge path, and the vendor tally actually dispatched.

## Token discipline

- Keep root context lean: gather, dispatch, relay. Put bulk reading and iteration loops in subagents.
- Judge-tier tokens cover the plan and the verdict; the mission's bulk work runs on cheaper tiers.
- At 30% context, stop opening phases or waves; finish in-flight work, give the verdict on what completed, and list what remains. The hard ceiling is 35%; split an oversized mission in Phase 1.
- This composes with `/low-then-rev`: it governs thinking effort; `/captain` governs who works and where judgment lands.
