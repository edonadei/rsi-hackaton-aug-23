# Cost, latency and engine

The phase-5 branch of [`SKILL.md`](SKILL.md): the two axes beside the score, and
how to choose the model the skill is measured on.

A skill's score says whether it works. Tokens and wall-clock say what it costs to
work, and both are already recorded on every run you have saved — no extra run is
needed to start reading them.

## Where the numbers are

- **Per task**: the report table carries a Tokens and a Wall column beside each
  task's score.
- **Per run**: the summary block prints `Tokens` as *in / out* and `Wall` with a
  per-attempt average taken over usable attempts only.
- **Between runs**: `caliper compare A B` prints token and wall deltas alongside
  the score delta — green when B is cheaper, red when costlier.

Deltas on those two columns **never** flag a regression. Only the score does. A
token rise at an equal score is a trade-off for you to weigh, not a failure the
tool will decide for you.

Read the four token fields as disjoint: `input_tokens` is non-cached prompt only,
with cache accounted separately in `cache_read_tokens` and
`cache_creation_tokens`, so the total never double-counts. A big cache-read
number is not waste — it is the same prompt arriving cheaply.

Watch the **unusable spend** line. Tokens and seconds burned by timed-out or
throttled attempts were really spent, but they say nothing about the skill; a
comparison whose entire cost movement sits there measured the weather.

## Editing for tokens

Every one of these is an ordinary candidate edit: state the hypothesis, make the
change, run, compare, keep or revert. None of them is exempt from the score.

- **Progressive disclosure.** Push reference out of `SKILL.md` into a sibling
  file behind a pointer. Caliper installs the whole skill directory and the
  relative links resolve, so a run genuinely measures whether the agent reaches
  the file when it needs it — the saving is real, and so is the risk of hiding
  something every branch needed.
- **Prune no-ops.** An instruction the model already follows by default costs
  tokens to say nothing. The test is whether behaviour changes without it, and
  the answer comes from an ablated run, not from an argument.
- **Collapse restatements into leading words.** A concept spelled out at three
  sites becomes one token repeated three times.
- **Cut duplication.** One meaning, one home.
- **Shorten the description.** It is loaded on *every* turn, whether the skill
  fires or not, so its tokens are the most expensive in the file. Watch the
  activation scoreboard when you trim it: this is the one edit that can cut
  tokens and quietly stop the skill firing at all.

## Editing for latency

Wall-clock is the agent thinking plus every tool call it makes. What moves it:

- **Fewer forced round trips.** A step that mandates a command the agent could
  have skipped costs a full turn every run.
- **Fewer strictly-sequential steps.** Steps that could be stated as one pass
  over the same material often are.
- **Less reading.** Disclosure cuts latency for the same reason it cuts tokens —
  on the branches that never open the file.

Two caveats before believing a wall-clock delta:

- Compare at the **same `--workers`**. Parallel attempts contend, and the
  per-attempt average moves with the setting.
- Wall-clock is the noisiest number Caliper reports — it moves with API load you
  do not control. Trust it at k≥5, and trust a 30% shift long before a 5% one.

## The non-inferiority bar

The rule for accepting a cheaper variant, and the reason a shortening pass does
not need to prove itself *better*:

> At **k≥5**, the candidate's score is within a small margin (≤5%) of the current
> head **and** still beats the ablated floor.

Equalling the head is a bonus, not a requirement. The claim being tested is *no
worse*, and the win is on the cost axis. Below the floor, there is no trade-off
to weigh — the skill has stopped earning its context.

## Choosing the engine

Backend and model are a **runtime axis**, never spec fields: `--model` for the
skill, `--judge-model` for the judge, chosen per run and recorded in the saved
result. That is what lets one spec outlive a model generation.

Backends: `claude-code`, `codex`, `pi`, `hermes`. A `--model` value takes
`backend:model`, a bare backend, or a bare model name.

To sweep:

```bash
for m in claude-opus-5 claude-sonnet-5 claude-haiku-4-5-20251001; do
  caliper run <spec> --k 5 --model "$m" \
    --judge-model claude-code:claude-sonnet-5 \
    --output ".caliper/runs/engine-$m.json"
done
caliper compare .caliper/runs/engine-claude-sonnet-5.json .caliper/runs/engine-claude-opus-5.json
```

**Hold the judge fixed across every arm.** Sweeping both at once produces a table
where no cell can be attributed, and it points self-preference bias straight at
the result — see [`RUBRICS.md`](RUBRICS.md).

What the sweep is for, in order:

1. **Where the skill breaks.** A skill that scores 0.9 on the frontier model and
   0.4 two tiers down is carrying its reliability on the model, not the text.
   The gap is a to-do list: the tasks that only the big model passes name exactly
   which steps are under-specified.
2. **The cheapest engine that clears the bar.** Same non-inferiority rule, with
   the model as the variable instead of the text.
3. **The floor's engine.** An ablated run on a stronger model raises the floor —
   a skill can stop earning its context simply because the base agent got better.

## Artificial Analysis as a prior

[artificialanalysis.ai](https://artificialanalysis.ai) publishes cross-model
intelligence indices, output speed (tokens/sec), time-to-first-token and price
per million tokens. Use it to **pick which models are worth a run** — a sweep is
expensive and the index narrows the candidate list to a few tiers apart.

Use it for nothing else. It scores general capability on public benchmarks; your
eval scores this skill on your tasks. A model that indexes higher can lose your
sweep, and when the two disagree, **your eval is the authority** — that
disagreement is a finding about your tasks worth writing down, not a reason to
doubt the run.

Record the backend, the resolved model and the judge model in the ledger entry.
A score without its engine is not reproducible.
