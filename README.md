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
skill does the job. **Activation rate** says whether it was the one that fired —
every spec installs all five skills, so an edit that broadens a skill until it
steals a neighbour's triggers shows up as a loss instead of hiding as a win.

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

## Status

The corpus and the scoreboard are done and validating. The climber is a stub —
that is the part still to write.
