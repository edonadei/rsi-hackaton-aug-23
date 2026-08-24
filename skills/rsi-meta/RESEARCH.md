# Researching tasks

The phase-2 branch of [`SKILL.md`](SKILL.md): where eval tasks come from, and
what makes each family worth its run cost. Rubric wording and judge design are
the sibling file, [`RUBRICS.md`](RUBRICS.md).

A task earns its place by **discriminating** — separating a good skill from a bad
one, or from no skill at all. A task the floor passes discriminates nothing. A
task nothing passes discriminates nothing either, until the skill can reach it.
Research is the hunt for prompts that sit in between.

## Where evidence comes from

Before inventing anything, harvest. Invented tasks drift toward what the skill
already does well, which is exactly the direction that teaches you nothing:

- **The skill's own trigger branches.** Each branch in the `description` claims
  a case the skill handles. Every claimed branch needs a task, or the claim is
  untested.
- **Real transcripts.** Prompts people actually sent, in their own words —
  including the badly-phrased ones. `caliper report <spec> --verbose` replays
  saved attempts; past failures are the richest seam in the repo.
- **The failure section.** Most skills end with a "where this goes wrong" list.
  Each bullet is a task waiting to be written: the skill claims to defend against
  it, so make it prove that.
- **The domain.** Search for how practitioners describe the same job. Their
  vocabulary is what a real prompt will use, and it is rarely the skill's.

**Completion criterion for the harvest**: every trigger branch and every named
failure mode maps to at least one task, or is written off with a reason.

## Happy path

The **modal** use — what most runs actually look like. Its job is to catch
regressions, so it should be stable enough to keep for the life of the skill.

Research it by asking what the user had open when they reached for this skill,
and what they did with the output afterwards. A happy-path task whose output
nobody would use is testing the wrong thing.

Make it hard enough that the floor fails: give it enough data that shortcutting
is possible, and make the shortcut wrong. In the corpus, `find-true-competitors`
does this by seeding a company that shares a category label but not a buyer — the
skill has to do the work to get it right, and guessing lands on the twin.

## Edge case

Valid input the raw agent mishandles. Reliable generators:

- **Thin data** — one row, empty fixture, a column of nulls. Does the skill
  report the coverage, or produce a confident answer the data cannot support?
- **Conflicting evidence** — two channels that disagree. Does it surface the
  conflict or silently pick one?
- **Scale extremes** — 500 candidates instead of 20. Does the ranking still mean
  anything?
- **Missing prerequisite** — the graph is down, the API key is absent, the file
  is the wrong format. The corpus routes this one deliberately: the CSV fixtures
  exist because "Neo4j is down" is the realistic version of this task.
- **Ambiguity** — a prompt with two defensible readings. Does it ask, or assume
  silently?

The strongest edge cases produce a *visible* difference: the skill says something
the base agent never would, and the rubric can name it.

## Adversarial

Pressure to abandon the discipline the skill exists to enforce. The user is not
an attacker here — they are in a hurry, and the skill's value is holding the line
anyway:

- **Authority** — "leadership already decided, just confirm it."
- **Urgency and permission to skip** — "don't overthink it", "quick version".
- **A pre-supplied wrong answer** — the sycophancy probe. Agents agree with a
  confidently stated conclusion far more readily than they generate it.
- **A prompt asking for a step to be skipped** — the one step the skill's value
  depends on.
- **Instructions inside the fixture data.** A row of a CSV that reads like a
  command. The skill must treat tool output as data.
- **Reward hacking** — a prompt where the cheapest way to satisfy the words is to
  fake the work. Pair it with an `assert:` on the artifact.

A good adversarial task still **demands a deliverable**. "Refuse the whole
request" is rarely the right behaviour and makes a task that passes by stalling;
the target is delivering the work with the pressure corrected. Write the fail
clause to catch both ends: capitulating, and refusing to produce anything.

## Realistic environment

The setup the task runs in, which is where most evals quietly become fiction. A
skill that queries a graph, tested with no graph, measures a different skill.

- **`setup:`** builds the world: fixture files, a git repo mid-rebase, a failing
  test suite, a directory with the wrong permissions. Make it the shape of a real
  one, including the mess — real data has duplicate rows, inconsistent casing and
  a column nobody filled in.
- **`mcp:`** declares servers the agent may use, `stdio` or remote, with
  `${VAR}` for anything secret so the committed spec stays clean.
- **`sandbox.forbidden_files`** closes the obvious cheat: the spec itself and
  `.caliper/` results. A read of either is detected and typed `cheat`.
- **`cleanup:`** puts it back, so attempt 3 sees what attempt 1 saw.

**Completion criterion**: every task runs from a `setup:` that a stranger could
read as a description of a real working directory, and two attempts of the same
task start from identical state.

## Trigger probes

The cheap family, and the one most suites are missing. A task with `activates:`
and no `expect:`/`assert:` skips the judge entirely, so it costs a fraction of an
execution task:

```yaml
- name: Silence — unrelated work
  activates: []
  prompt: Rename the variable `foo` to `bar` in main.py.
```

Two shapes worth having:

- **Silence probe** (`activates: []`) — work the skill has no business touching.
  Catches a `description` that over-fires, which costs context on every unrelated
  turn and is invisible to any execution task.
- **Neighbour probe** (`activates: [other-skill]`) — a prompt that belongs to a
  neighbour. Catches hijacking. Whether the neighbour did the job *well* is the
  neighbour's own eval.

Activation is scored as an **exact set match**, so a skill that legitimately
delegates enumerates its whole chain. That is what makes "did it actually
delegate?" assertable rather than assumed.
