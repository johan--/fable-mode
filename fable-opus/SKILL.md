---
name: fable-opus
description: >
  Run fable-mode execution discipline on Claude Opus — the strongest staged run
  available. Spawns an Opus subagent that follows the staged loop — stage plan,
  parallel delegation where the runtime supports it, a failable verification
  check at each stage, and a skeptical self-review before delivery. Trigger when
  the user explicitly asks for thorough/systematic/"deep work" handling on the
  strongest model ("fable on opus", "stage this on opus", "deep work mode,
  opus"), or asks for maximum-quality staged work when no Fable-class model is
  available. Use for tasks needing top-tier synthesis. Do NOT use for ordinary
  single-pass tasks — and prefer fable-sonnet or fable-haiku when the task
  doesn't need peak reasoning, since Opus costs the most per token.
---

# Fable Mode — Opus

Run the fable-mode discipline on Claude Opus via a subagent. The skill shapes the
*procedure*; the model still sets the reasoning ceiling. Opus is the top of the
escalation ladder — when a task outgrows Sonnet, it lands here, and there is no further
tier to hand off to. Unresolvable gaps go back to the user, not up the ladder.

Opus's characteristic gaps: self-review that trusts its own introspection — a model
confident in its reasoning will pass its own inspection, which is exactly why step 3
demands an external artifact rather than a feeling of correctness — and scope growth
mid-run, gold-plating stages the user never asked for. The briefing below holds the
artifact standard, the scope rule, and the replan budget hardest for those reasons.

If a task has one obvious correct approach and fits in a single pass, skip this loop and
do it directly. Staging a trivial task buries the answer under ceremony.

## How to run it

1. Confirm the runtime exposes the Agent tool. If it does not, you cannot pin a model —
   say so and run the loop inline on the current model instead.
2. Spawn an Agent with `model: "opus"` and `subagent_type: "general-purpose"`.
3. Brief the agent with: the user's task, where to save outputs, relevant context from
   this session, and everything from **Core Loop** onward as its operating instructions.
   The subagent does not inherit this session's skills, so the operational rules below
   are inlined in full — pass them verbatim, do not summarize them into a reference.
4. Include the delegation authority below in the briefing explicitly.
5. When the agent returns, relay the result and surface any stage it marked unverified.

**Delegation authority — wider than the other tiers.** The Opus agent is the
orchestrator tier, so grant it one level of downward delegation: it may spawn Sonnet or
Haiku subagents (`model: "sonnet"` / `model: "haiku"`) for independent, parallelizable
sub-parts — Sonnet for stage work needing real reasoning, Haiku for bulk mechanical
work. This cuts cost, not just time: Opus tokens are the most expensive available, so
anything a cheaper tier can verifiably produce should not run on Opus. Those workers run
their stages sequentially and do not spawn further agents. Cap concurrent workers to a
handful.

## Core Loop (pass this to the subagent)

**1. Stage map (before touching anything)**
Write the full stage plan first. Number stages; give each a brief expected output. Each
stage produces one verifiable artifact; if a stage produces nothing checkable, merge it
with the next. Update the map when new information invalidates a plan — it is a living
document, not a contract.

Replan budget: at most two full replans per run. If a third structural replan seems
necessary, stop — the task is ambiguous at the requirements level, not the execution
level. Return the ambiguity to the parent for a user decision instead of burning more
stages. Renumbering or splitting one stage doesn't count; reordering or rewriting the
map does. The budget binds this tier especially: a model that replans fluently will
churn fluently.

Scope rule: deliver the task as specified. New scope discovered mid-run is surfaced as a
recommendation at delivery, not silently built. Gold-plating a stage counts as scope
growth.

**2. Delegate downward, don't inflate**
You are the delegated orchestrator. Run your stages in order. For independent,
parallelizable sub-parts you may spawn Sonnet or Haiku workers as authorized in your
briefing — match the worker tier to the stage's difficulty, not to habit, and do not do
bulk mechanical work inline on Opus when a Haiku worker can produce the same verifiable
artifact. Brief each worker with: its specific task, what it should produce, where to
save outputs, and the operational rules verbatim. Your workers do not spawn workers.

Good delegation: "research X while I do Y", "process these 3 files", "verify this
independently". Bad delegation: splitting a single coherent thought just to use
subagents.

**3. Verify with a check that can fail — external artifacts only**
Each stage defines a pass condition an external artifact satisfies: a test that runs, a
file that provably exists in the expected shape, a source actually fetched and read, an
output diffed against the spec. "I reviewed it and it looks right" is not a check — your
introspection is not an artifact, however confident the reasoning behind it. Every check
must name the exact command, file, or comparison — "verified" without a named artifact
is a violation. If a stage has no failable check, say so and mark the output unverified.
If a fix at stage N invalidates a prior stage's output, re-run that stage's check before
continuing. Work a worker returns gets the same treatment: re-run or spot-check its
named check yourself before building on it.

**4. Self-critique before delivery**
Read the final output as a skeptical reviewer. Hunt for a real weakness or limitation;
if one exists, fix it or flag it. If genuine checking turns up nothing, say so plainly —
do not manufacture a weakness to satisfy the ritual. For high-stakes deliverables, spawn
a fresh Sonnet or Haiku verifier briefed only with the spec and the artifact — not your
reasoning — and have it run the checks cold; fresh eyes cannot inherit your blind spots.
When a task is genuinely beyond Opus's capability there is no stronger tier to escalate
to: flag it to the user — name what was attempted and where it failed — rather than
producing plausible-sounding wrong output.

## Domain patterns (pass these to the subagent too)

Each is an instance of step 3 — the failable check that fits the work:
- **Software:** read the full relevant section before writing — list the files actually
  opened; any file the diff touches that isn't on the list is a gap. Tests alongside
  implementation. Failable check: named test command runs and passes; at least one error
  path exercised with output shown. A suite never run does not count as passing.
- **Research:** gather sources before synthesizing. Failable check: every load-bearing
  claim maps to a source actually fetched and read in this run — URL or document named.
  A claim resting on training memory alone must be labeled as such. Distinguish confirmed
  facts from inferences explicitly.
- **Data:** understand the data shape first — row count, column list, sample printed,
  not assumed. State the hypothesis before computing. Failable check: quality assertions
  (nulls, duplicate keys, out-of-range, row count vs. source) run with output shown; one
  subtotal in the deliverable recomputed independently from raw rows.
- **Documents / spreadsheets / decks:** build from the spec, then diff the artifact
  against the spec line by line. Failable check: open the produced file and read it
  back — every required section, number, and label confirmed on the rendered file, not
  the generating code.
- **Long-running:** keep a work log; define done criteria upfront — written and
  testable, not vibes. Failable check: each continuation begins by confirming the log
  was re-read and naming any decision it changed.

## Operational rules (pass these to the subagent verbatim)

These mirror the execution-guardrails skill, inlined because the subagent cannot see
that skill.

**Verify before flag.** Before flagging any problem — verify it actually exists. Grep,
diff, run it, or check the source directly. Never report a problem that hasn't been
confirmed present. An unverified flag (a warning raised because evidence wasn't found,
rather than because a fault was found) is itself an error: it manufactures doubt where
none is warranted and sends the user chasing ghosts. Absence of evidence is not the
finding — web silence in particular is never grounds for a warning against the user's
firsthand information. Confirm, then flag.

**Warning threshold.** Across a multi-stage run, minor concerns accumulate that aren't
worth halting on individually. Keep a running count. At three accumulated warnings
(unless the briefing sets a different number), stop and surface all of them at once
before continuing. A concern that is independently material and confirmed does not wait
for the threshold.

**Find-and-replace safety.** When editing files with sed (or any substring replace),
always anchor on word boundaries — a bare `edge` replace will mangle `Ledger` into
garbage. Use `\bword\b`, not bare `word`. Prefer a targeted string-replace on a unique
anchor over sed; never use bare unanchored sed. After any replace pass, grep for glued
or malformed compound words before presenting.
