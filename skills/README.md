# Skills

Five unrelated workflow skills, plus the meta-framework that improves them.

The five are the **corpus**: deliberately independent of each other, each with an
eval spec beside it, so a change to one is measurable on its own. They are what
`meta-framework-rsi-skill-improvement` hill-climbs on — the subject matter is
incidental, the score is the point.

| Skill | Workflow |
| --- | --- |
| `dynamic-query-neo4j` | Query a Neo4j graph without guessing its schema |
| `find-true-competitors` | Separate real substitutes from category twins |
| `optimize-keyword-strategy` | Rank keywords by winnability and cluster them into pages |
| `find-reddit-relevant-threads` | Pick Reddit threads worth entering |
| `optimize-parameter-next-video-script` | Set the next video's parameters from past performance |
| `meta-framework-rsi-skill-improvement` | The hill-climbing loop itself — placeholder |

## Install

```bash
npx skills add edonadei/rsi-hackaton-aug-23
```

Add `--all` to take every skill without prompting, or `--skill find-true-competitors`
for one. `--global` installs at user level instead of into the current project.

```bash
npx skills add edonadei/rsi-hackaton-aug-23 --list
```

## Evals

Each skill carries a `<name>.eval.yaml` beside it, with a happy path, an edge
case, and an adversarial task. Tasks feed the agent CSV fixtures rather than a
live database, so they run anywhere.

```bash
caliper run skills/find-true-competitors/find-true-competitors.eval.yaml --k 3
```

Each spec installs only its own skill and asserts `activates:`, so a run measures
two things: whether the skill does the job, and whether the agent reached for it
at all. The five are unrelated, so there is no neighbourhood worth installing.

`--baseline` scores the raw agent on the same tasks — the floor any edit has to
beat. `caliper compare <before> <after>` diffs two saved runs task by task, which
is the measurement the hill-climb runs on.

## Graph labels

Several skills query Neo4j. Every label and relationship they name (`:Company`,
`:Keyword`, `:TARGETS`) is a **placeholder** — the skills read the live schema at
run time rather than trusting those names.
