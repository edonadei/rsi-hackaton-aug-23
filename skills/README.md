# Skills

Growth-intelligence skills over the project's Neo4j graph.

`dynamic-query-neo4j` is the shared foundation — the other four invoke it for
connection handling and schema recon before touching the graph.

| Skill | Answers |
| --- | --- |
| `dynamic-query-neo4j` | What does the graph actually say? |
| `find-true-competitors` | Who do buyers pick instead of us? |
| `optimize-keyword-strategy` | Which terms are worth going after next? |
| `optimize-parameter-next-video-script` | What should the next video's parameters be? |
| `find-reddit-relevant-threads` | Which threads are worth entering? |

## Graph schema

Every label and relationship named in these skills (`:Company`, `:Keyword`,
`:TARGETS`) is a **placeholder**. The skills read the live schema at run time
rather than trusting those names — substitute what recon finds. When the graph's
vocabulary settles, record it here and the recon step gets cheaper.

## Evals

Each skill carries a `<name>.eval.yaml` beside it. Every spec installs all five
skills, so a run also measures whether the right one fires rather than a
neighbour with overlapping vocabulary.

```bash
caliper run skills/find-true-competitors/find-true-competitors.eval.yaml --k 3
```

Add `--baseline` to check the skill beats the raw agent. The eval tasks feed the
agent CSV fixtures rather than a live database, so they run anywhere.
