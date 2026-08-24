---
name: rsi-meta
description: Hill-climb a skill against its eval — harden the rubric, measure the ablated floor, then edit, re-run and keep only what the numbers justify. Use when asked to improve, optimise or tune a skill, to strengthen an eval or its rubrics, to ablate a skill, to cut its token cost or latency, to pick which model it runs best on, or to version it after a measured win.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebSearch, WebFetch
---

# rsi-meta

A skill is a document, and a document can be rewritten. What separates
**improvement** from churn is a score you believe in — so this loop guards the
scoreboard before it climbs it.

The whole loop is one discipline: **move one thing per measured step.** Change
the eval, or change the skill, never both between two runs, or the delta belongs
to nothing you can name. Every phase below ends in a saved run, and every saved
run is reachable by path from the ledger in phase 6.

Requires `caliper` (`pipx install caliper-eval`). Every command in this file is
one the loop actually issues, on current flags: the floor is measured with
`--ablate` and read with `caliper compare`, which is the pair that replaced the
retired `--baseline`.

## 1. Pin the run

Name, out loud, the five things a number is meaningless without:

- **Subject** — the `SKILL.md` under test, by path.
- **Spec** — the `*.eval.yaml` beside it.
- **Engine** — `--model` for the skill and `--judge-model` for the judge, chosen
  at run time and never written into the spec.
- **k** — attempts per task. `--k 1` debugs a spec, `--k 3` reads a direction,
  `--k 5` decides a keep-or-revert.
- **Head version** — the top entry of `VERSIONS.md` beside the skill, or *none*
  if this is the first climb.

**Completion criterion**: `caliper validate <spec>` passes and all five are
stated. A climb that starts on an invalid spec measures the harness, not the
skill.

## 2. Harden the scoreboard

Evals-first, because a weak eval makes every later number a lie: the loop will
happily climb it, and the skill will get worse in the ways the eval cannot see.

Read [`RESEARCH.md`](RESEARCH.md) and build out the four task families — happy
path, edge case, adversarial, realistic environment. Read [`RUBRICS.md`](RUBRICS.md)
and rewrite every `expect:` to name its required evidence and its fail clause,
with the judge's own biases designed around rather than hoped away.

Then split the suite in two. The spec you climb is the one you **overfit**; the
holdout is the one that tells you so:

- `<name>.eval.yaml` — the climbing set. Read on every iteration.
- `<name>.holdout.eval.yaml` — same families, different instances. Run at the
  start and once more before shipping, never in between.

A candidate that gains on the climbing set and loses on the holdout is
memorisation, not improvement. Revert it.

**Completion criterion**: every family present in both specs, every `expect:`
carrying evidence and a fail clause, at least one deterministic `assert:`, both
specs validating, and the eval's own version bumped in the ledger. The skill is
untouched in this phase.

## 3. Measure the floor

The **floor** is what the bare agent scores on these tasks with the skill
removed. Anything at or below it is context spent for nothing.

```bash
caliper run <spec> --k 5 --ablate <skill-name> --output .caliper/control/<skill-name>-ablated.json
```

An ablated run is a property of the tasks and the surviving neighbourhood, never
of the removed skill's text — so it is run **once** and re-compared against every
later iteration. Pin it by path, because `caliper compare` reads a bare spec name
as that spec's *latest* run and both arms live in one folder.

A task the floor already passes is a finding about the **task**: it is too easy
to separate a good skill from no skill. Rewrite it harder or retire it.

**Completion criterion**: the control JSON saved at a stable path, its per-task
scores read, and every task the floor passes either rewritten or retired with the
reason recorded.

## 4. Climb

One hypothesis per step, stated before the edit:

1. **Name the hypothesis** — "the step-3 completion criterion is too vague, so
   the agent stops early". One claim, one edit, one measurement.
2. **Edit** the skill. Nothing else moves — not the spec, not the engine, not k.
3. **Run** it: `caliper run <spec> --k 5 --output .caliper/runs/<version>.json`
4. **Compare**: `caliper compare .caliper/runs/<previous>.json .caliper/runs/<version>.json`
5. **Decide**, by the rule below, and write the ledger line either way.

| Outcome | Decision |
| --- | --- |
| Score up, no matched task below | **Keep** |
| Score flat, tokens or wall down | **Keep** — see [`COST.md`](COST.md) |
| Any matched task below | **Revert**, and record what the edit cost |
| Only unusable attempts moved | **Re-run** — that is judge or infra noise, not the skill |

Read **which scoreboard moved** before writing the next hypothesis. They have
opposite fixes and never merge:

- **Activation** dropped — the agent stopped reaching for the skill. The bug is
  in the `description`, and the fix is a sharper trigger branch.
- **Execution** dropped — the agent reached for it and still failed. The bug is
  in the body, and the fix is a sharper step or completion criterion.

Writing craft for the edits themselves — completion criteria, leading words,
progressive disclosure, pruning — belongs to `writing-for-agents`. Invoke it
rather than restating it; this loop supplies the measurement, that one supplies
the prose.

**Completion criterion**: every candidate edit ends in keep or revert, each with
a saved run pair and a ledger line naming hypothesis, delta and decision. No edit
is left unmeasured, and no edit is kept on the strength of how it reads.

## 5. Cost and engine

Score is not the only axis. Two more decide whether a skill is worth its context,
and both are read from runs you have already saved: **tokens** and
**wall-clock**. Which model to run it on is the third.

[`COST.md`](COST.md) carries all three — where the numbers live, which edits
actually move them, the non-inferiority bar for accepting a cheaper variant, and
how to sweep `--model` across backends without turning the judge into a variable.

**Completion criterion**: the shipped version has a token and wall figure beside
its score, and a named engine it was measured on.

## 6. Version

A version is a **recipe**, not a number: the text, the score, and everything the
score depended on. `VERSIONS.md` lives beside the skill, newest first:

```markdown
## v4 — 2026-08-23
- **Hypothesis**: step 3's completion criterion was vague; the agent stopped early.
- **Change**: bound it to "every candidate from step 2 promoted or cut with a reason".
- **Score**: 0.87 (v3: 0.73, floor: 0.40) · **Activation**: 1.00
- **Cost**: 41k tokens · 38s per attempt
- **Engine**: claude-code / claude-opus-5, judge claude-code / claude-sonnet-5, k=5
- **Runs**: `.caliper/runs/v4.json` vs `.caliper/runs/v3.json` · floor `.caliper/control/<name>-ablated.json`
- **Holdout**: 0.80 (v3: 0.80)
```

Reverted candidates get an entry too, marked `— reverted`. The record of what
failed is what stops the loop re-trying it in three iterations' time.

**Completion criterion**: the head entry's runs resolve to files on disk, its
engine is named, and its score is reproducible by re-running the named spec at
the named k.

## Stopping

The loop ends on any of these, and says which:

- Three consecutive candidates fail to beat the current head. The remaining gap
  is in the tasks, not the text — go back to phase 2.
- The climbing set is saturated (every task at 1.0) while the holdout is not.
  The eval is exhausted; write harder tasks.
- The score matches the floor after a genuine effort. The skill is not earning
  its context, and saying so is the finding.

## Where this goes wrong

- **k too low.** A 2/3 and a 3/3 differ by one coin flip. Decide at k≥5.
- **Judge noise read as regression.** `infra_error`, `timeout` and `judge_error`
  are *unusable* and leave the denominator; a comparison whose only movement is
  in that column measured nothing.
- **Both halves moved.** An edit to the skill and the spec between two runs
  produces a delta that belongs to neither.
- **Cross-era or cross-neighbourhood comparison.** Two runs that installed
  different skills, or ablated different ones, are refused for a reason.
- **The transcript read as the artifact.** A confident write-up is not evidence
  that the work happened; grade the artifact wherever one exists.
