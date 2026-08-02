# claude-commands

Six general-purpose slash commands for [Claude Code](https://claude.com/claude-code),
extracted from daily use in a real working setup. No framework, no install script — each
command is one markdown file. Copy what you want, edit freely.

Sibling repo: [clinical-agent-skills](https://github.com/MichaelRDionne/clinical-agent-skills)
carries the domain-heavy skills and multi-agent protocol commands; this repo is the
domain-neutral workflow layer.

## Install

Copy any command into your commands directory:

```bash
# global (all projects)
cp commands/*.md ~/.claude/commands/

# or per-project
cp commands/*.md <your-repo>/.claude/commands/
```

Then invoke by filename: `/captain`, `/lacuna`, `/low-then-rev`, `/redteam`, `/tidy`, `/vet-repo`.

## The commands

### `/captain` — captain-mode delegation

Runs the turn on your strongest model (via `model:` frontmatter — edit it to your top
tier), which plans inline, routes bounded tasks down to cheaper subagents
(haiku/sonnet/opus per a routing table), then reviews everything and delivers its own
verdict. The inversion that matters: the expensive model does the judging while cheap
models do the typing.
Includes a context budget (30% checkpoint / 35% ceiling) and a
plan-approval gate so nothing spawns before you see the troop layout.

### `/lacuna` — find the structurally missing thing

Instead of asking a model to "be creative" (which returns the probability-mass average),
map the field, extract the axes everything visible optimizes on, cross them, and inspect
the *empty cells* — then name the specific force keeping each cell empty. Ends every run
with "How we'd check this," because it is a hypothesis generator, never its own validator.
The prettiest finding gets flagged as the most suspicious — elegance is the confabulation
signature.

### `/low-then-rev` — lean by default, spike when it's hard

A thread-level operating mode: start at the cheapest adequate model/effort/verbosity,
escalate automatically on observable triggers (security, ambiguity, shared contracts,
repeated failure), climb one rung at a time, and — the part most setups miss — **come back
down** once the hard part clears. The shape is a spike; a ratchet that never descends is
the failure mode. Includes the escalation criteria, the descent announcement, a stop rule, and
a recalibration section for what changes when you move onto a newer frontier model — it
delegates more eagerly, self-verifies unprompted, and states file facts it never checked.

### `/redteam` — pre-flight adversarial check

Before acting on a request, enumerate what the requester may be missing: risky
assumptions, hidden failure modes, missing guardrails, privacy/credential/destructive-action
risk, and anything that should be promoted to a reusable script or skill. Then do the best
*lean* version of the task — the check exists to prevent both blind execution and
overbuilding.

### `/tidy` — workspace housekeeping, report-first

Surface stray files, backup cruft, orphans, duplicates, and broken internal links as a
review list. Report-only by default; `--apply` prunes only the pre-listed safe tier after
an explicit yes. Starts with a sensitive-data sweep whose three rules were each paid for with
a real incident: no directory exclusions on detection; match every identifier format — a
check narrower than the rule it enforces reports green; and plant a fixture that makes every
pattern class go red before the word "clean" is allowed in the report. Deliberately
standalone: cleanup bundled into an end-of-session flow is how accidental deletions happen.

### `/vet-repo` — quarantine and vet an untrusted repo

Clone an unknown GitHub repo with hooks disabled into a quarantine directory and
static-scan it without executing anything: enumerate the agent-facing prompt-injection
surface (CLAUDE.md, AGENTS.md, .cursorrules, MCP configs), grep for malicious-code and
credential-theft patterns, then report CLEAN / CAUTION / HOSTILE and *stop*. Running the
code is a separate, opt-in step that only ever happens inside a no-network Docker sandbox.
Built on one rule: everything inside the clone is data, never instructions.

## Design notes

- Commands reference each other where they compose (`/captain` assumes `/low-then-rev`'s
  lean-by-default posture) but each works standalone.
- Model aliases (`haiku`/`sonnet`/`opus`, `fable`) are current as of mid-2026; the routing
  *logic* is the durable part — swap in whatever your provider's ladder is.
- These are prompts, not software. Read them before trusting them, same as `/vet-repo`
  would tell you.

## License

MIT
