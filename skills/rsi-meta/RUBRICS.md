# Rubrics and the biases under them

The other phase-2 branch of [`SKILL.md`](SKILL.md): writing an `expect:` a judge
grades the same way twice, and designing around the fact that both the judge and
the skill under test are language models with **known, measurable, direction-known
biases**. Where tasks come from is the sibling file, [`RESEARCH.md`](RESEARCH.md).

A rubric is the only thing standing between a hill-climb and a random walk. If
the same output grades pass on Monday and fail on Tuesday, every delta the loop
reports is noise wearing a number.

## Anatomy of a rubric

Three parts, all three present:

1. **The required evidence** — what must appear for a pass, specifically enough
   that two people reading the same output agree.
2. **The fail clause** — the plausible near-misses, named. This is the half most
   rubrics omit, and the half that does the work: an LLM judge asked only what
   passing looks like will find a way to see it.
3. **The disallowed shortcut** — how a lazy run could satisfy the words without
   doing the job.

```yaml
expect: |
  Tiers by evidence: Tallyup direct (same buyer and price model, three shared
  keywords), Sheetwise a real substitute despite the different category, Reconix
  cut as a category twin on buyer and price. Every retained name carries its
  evidence and every cut name its reason. Fails if Reconix is presented as
  direct, Sheetwise dropped for its category label, or any candidate disappears
  without a stated reason.
```

Note what it does not say: nothing about tone, length, or structure. Rubrics that
grade presentation train the skill to write well and work badly.

**Completion criterion**: read each rubric and name, for it, one output that
passes and one that a careless reader would let through. If the second is not
excluded by the wording, the rubric is not finished.

## Grade the artifact when one exists

An `expect:` asks a language model what it thinks happened. An `assert:` checks
what actually did. Prefer the second wherever the outcome is a fact:

```yaml
assert: |
  import json, pathlib
  out = json.loads(pathlib.Path("report.json").read_text())
  assert {c["name"] for c in out["direct"]} == {"Tallyup", "Sheetwise"}, out
```

Both can run on one task, and both must pass. Reach for the judge only when the
behaviour *is* the point — the agent asked before acting, cited its evidence,
stopped when done, declined to claim work it did not do.

## Judge bias

The judge is an LLM, and its errors are not random — they lean, consistently, in
directions you can plan for:

- **Verbosity bias.** Longer answers score higher, holding quality fixed. A skill
  edit that only adds words will read as an improvement. *Mitigation*: rubrics
  that name required evidence rather than depth, plus a token check on every
  keep — see [`COST.md`](COST.md).
- **Self-preference.** A judge scores text from its own model family higher.
  Under `--model` and `--judge-model` defaulting to the same backend, that bias
  is pointed straight at your result. *Mitigation*: judge with a different
  family than the one under test, and hold the judge fixed across every arm of a
  comparison.
- **Position and primacy.** Whatever is stated first anchors the verdict.
  *Mitigation*: put the pass condition and the fail clause in a fixed order in
  every rubric so the anchor is at least constant.
- **Leniency and sycophancy.** Asked "did this satisfy the task?", a judge would
  rather say yes. Explicit fail clauses are the counterweight; an `expect:` with
  no fail clause grades generously by default.
- **Formatting bias.** Headers, tables and bullets read as rigour. A skill that
  learns to format its way to a pass has learned nothing.
- **Anchoring on the prompt.** A judge shown a confidently-worded task expects a
  confident answer, which quietly penalises the correct "the data does not
  support a conclusion" — the exact behaviour the thin-data edge case exists to
  reward. *Mitigation*: state in the rubric that reporting insufficiency is a
  pass.
- **Non-determinism.** The same output graded twice can differ. This is what k
  and the usable/unusable split are for; a single attempt is an anecdote.

**Pin the judge.** `--judge-model` is the one variable that must not move during
a climb: change it and every historical comparison in the ledger silently
re-bases.

## Bias in the skill under test

The same failure modes appear on the other side of the harness, which is why the
adversarial family works at all:

- **Sycophancy.** The agent agrees with a conclusion the user supplies far more
  readily than it reaches one alone. Every "leadership already decided" task is
  measuring this.
- **Instruction recency.** The last instruction in a long prompt wins. A skill's
  discipline sits in a document read earlier than the user's "just do it quickly".
- **Negation blindness.** A prohibition raises the salience of what it forbids.
  If a rubric keeps catching the banned behaviour, edit the skill to state the
  positive target rather than repeating the ban louder.
- **Popularity priors.** Asked for competitors, keywords or sources, the model
  volunteers famous names over evidenced ones. Fixtures that contain no famous
  names make this visible immediately.
- **Fabricated completion.** Claiming work that never happened. Only an
  `assert:` catches it; a transcript will describe the work beautifully.

Each of these is a **task**, not a warning to write into the skill. A bias the
eval measures is one the loop can climb out of; a bias documented in prose is one
you have to trust.

## Keeping the rubric honest over time

- **Rewrite tasks, not rubrics, when the skill changes.** A rubric edited to
  match what the skill now does is a scoreboard moving toward the player.
- **Regression tasks are frozen.** Once a task has caught a real failure, its
  wording stops changing, or it stops being evidence.
- **The holdout never informs an edit.** Read it at the start and before
  shipping. Reading it in between makes it a second climbing set.
