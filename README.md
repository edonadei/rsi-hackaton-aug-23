# rsi-hackaton-aug-23

Built for **[Recursive Self Improvement Hack: Evals](https://luma.com/rkum5o5l)** — a
Sundai hackathon in San Francisco, 23 August 2026.

The premise of the day: a self-improving loop is only as good as its evaluator.
So this project is built **evals-first** — the measurement exists before the
thing that optimises against it.

## The idea

An agent skill is a document, and a document can be rewritten. If you can score
a skill, you can hill-climb it: edit, re-score, keep what wins. The score is the
hard part, so that is what got built first.

- **The corpus** — five workflow skills in [`skills/`](skills/), deliberately
  unrelated to each other so a change to one is measurable on its own.
- **The scoreboard** — a [caliper](https://github.com/edonadei/caliper) eval
  spec beside each skill: a happy path, an edge case, and an adversarial task.
  Fixtures are CSV, so no live database is needed.
- **The climber** — `meta-framework-rsi-skill-improvement`, the loop that reads
  the scores and rewrites the skills. **Currently a placeholder.**

## The loop

```
baseline  →  edit the SKILL.md  →  caliper run --k 3  →  caliper compare  →  keep or revert
```

Two numbers come back per run, and both matter. **Task pass@k** says whether the
skill does the job. **Activation rate** says whether the skill fired at all — a
skill the agent never reaches for scores nothing, however good its content.

Each spec installs only the skill under test. The five are unrelated, so a
neighbourhood would add cost and noise without measuring anything real.

`--baseline` scores the raw agent on the same tasks. That is the floor: a skill
that does not beat it is costing context for nothing.

## Try it

```bash
npx skills add edonadei/rsi-hackaton-aug-23
```

```bash
caliper run skills/find-true-competitors/find-true-competitors.eval.yaml --k 3 --baseline
```

Needs `caliper` (`pipx install caliper-eval`). Per-skill detail is in
[`skills/README.md`](skills/README.md).

## Tooling

The [caliper repo](https://github.com/edonadei/caliper) ships two skills that
drive this loop, so the work is done in conversation rather than by hand:

- **`grill-skill`** — interviews you into an eval, writes the `.eval.yaml`, then
  runs the create → test → improve loop. Every spec in `skills/` came from it.
- **`evaluate-skill`** — measures an existing skill: pass@k over k runs, a
  baseline comparison against the raw agent, and reads of the results.

```bash
npx skills add edonadei/caliper
```

They are also the reference implementation of the hand-run version of the loop.
`meta-framework-rsi-skill-improvement` is the attempt to close it — same steps,
no human in the middle.

## Status

The corpus and the scoreboard are done and validating. The climber is a stub —
that is the part still to write.
