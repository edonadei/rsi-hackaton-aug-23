---
name: dynamic-query-neo4j
description: Query the Neo4j graph by reading the live schema before writing Cypher. Use when a question needs graph data, when writing or debugging Cypher, or when a query comes back empty or with rows that look wrong.
---

# Dynamic query: Neo4j

The graph's schema is the source of truth, and it moves. Labels get renamed, relationships get a direction flipped, a property that was `name` last week is `title` today. Cypher written from memory fails **silently**: a wrong label matches nothing and Neo4j returns zero rows without an error, which reads exactly like "no data exists".

So every query starts with **recon**: read the schema from the live database, then write against what is actually there.

## 1. Connect

Find the connection before assuming one. In order, stop at the first hit:

1. `NEO4J_URI` / `NEO4J_USERNAME` / `NEO4J_PASSWORD` in the environment or a `.env` file at the repo root.
2. A configured Neo4j MCP server (check the available tools for one that runs Cypher).
3. `docker ps` for a running `neo4j` container, whose bolt port is usually `7687`.

When none of the three hits, stop and ask the user for the URI and credentials. Report which of the three you checked.

**Completion criterion**: a trivial query (`RETURN 1`) returns a row.

## 2. Recon the schema

Run all four, every session — cheap, and they replace guessing:

```cypher
CALL db.labels();
CALL db.relationshipTypes();
CALL db.propertyKeys();
CALL apoc.meta.schema();
```

When APOC is absent, get the shape of each label that matters from the data:

```cypher
MATCH (n:Label) RETURN n LIMIT 3;
MATCH (n:Label)-[r]-(m) RETURN type(r), labels(m), count(*) ORDER BY count(*) DESC;
```

That second query is the one that pays: it shows which relationships exist **and their real direction**, which is where hand-written Cypher usually goes wrong.

**Completion criterion**: for every label and relationship your question touches, you have seen a real sample row — the property names you are about to filter on, and which way the arrow points.

## 3. Write the query

- Filter on properties you saw in step 2, spelled as you saw them.
- Match undirected (`-[r:REL]-`) while exploring; pin the direction once you know it.
- Compare text with `toLower(n.name) CONTAINS toLower($term)` rather than `=`, so casing and stray whitespace stop costing you rows.
- Parameterise every user-supplied value (`$term`), never string-concatenate it into Cypher.
- `LIMIT 25` on exploratory queries. Aggregate before you return when the row count could run large.
- Read-only by default. Any `CREATE`, `MERGE`, `SET`, or `DELETE` gets confirmed with the user first, with the exact statement and the number of nodes it will touch.

Recipes for overlap, similarity, and ranking queries — the shapes the competitor, keyword, Reddit, and video skills all need — are in [`CYPHER-PATTERNS.md`](CYPHER-PATTERNS.md).

## 4. Verify before reporting

Zero rows is a result **and** a symptom. Before reporting "no data", strip the query back one clause at a time until rows appear, and report which clause emptied it. An empty result from a correct query and an empty result from a typo look identical, and only this step separates them.

Then sanity-check what came back: do the counts have a plausible magnitude, are the entities the ones the user meant, is a suspiciously round number hiding a `LIMIT`?

**Completion criterion**: you can state, for every number you report, the query that produced it. Show the Cypher alongside the answer so the user can check it.

## Reporting

Give the answer first, then the Cypher, then the caveats — which nodes were missing, which joins were sparse, where the graph's coverage runs thin. A number from a graph with three of the twenty expected nodes needs that stated next to it.
