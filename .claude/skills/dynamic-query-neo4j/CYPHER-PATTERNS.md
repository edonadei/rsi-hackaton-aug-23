# Cypher patterns

Query shapes the competitor, keyword, Reddit, and video skills reach for. Every label and relationship below is a **placeholder** — substitute what step 2 of `SKILL.md` actually found in the graph.

## Overlap between two entities

How much two nodes share, by way of a common neighbour. The backbone of competitor detection and keyword gap analysis.

```cypher
MATCH (a {name: $a})-[:TARGETS]->(k)<-[:TARGETS]-(b)
WHERE a <> b
RETURN b.name AS other, count(DISTINCT k) AS shared, collect(k.term)[..10] AS examples
ORDER BY shared DESC LIMIT 25
```

Raw counts favour whichever node has the most edges. Normalise with Jaccard when you rank on the result:

```cypher
MATCH (a {name: $a})-[:TARGETS]->(k)
WITH a, collect(DISTINCT k) AS ak
MATCH (b)-[:TARGETS]->(k2) WHERE b <> a
WITH a, ak, b, collect(DISTINCT k2) AS bk
WITH b, size([x IN ak WHERE x IN bk]) AS shared, size(ak) + size(bk) AS total
WHERE shared > 0
RETURN b.name, shared, round(1000.0 * shared / (total - shared)) / 1000 AS jaccard
ORDER BY jaccard DESC LIMIT 25
```

## Gap: what they have and you do not

```cypher
MATCH (rival)-[:TARGETS]->(k)
WHERE rival.name IN $rivals
  AND NOT EXISTS { MATCH (:Company {name: $me})-[:TARGETS]->(k) }
RETURN k.term, count(DISTINCT rival) AS rivals_covering, k.volume, k.difficulty
ORDER BY rivals_covering DESC, k.volume DESC
```

`rivals_covering` is the signal: a term every rival holds and you do not is a structural gap, while one rival holding it may be their idiosyncrasy.

## Clustering by shared neighbour

Group keywords or threads that hang off the same topic, so output is clusters rather than a flat list.

```cypher
MATCH (k:Keyword)-[:ABOUT]->(t:Topic)
RETURN t.name AS cluster, count(k) AS size,
       collect({term: k.term, volume: k.volume})[..15] AS members
ORDER BY size DESC
```

With no topic layer in the graph, cluster on shared token or shared head term in your own reasoning and say that is what you did.

## Ranked performance with a comparison baseline

For "which of my things worked", the per-row number means nothing without the cohort average beside it.

```cypher
MATCH (v:Video)
WITH avg(v.views) AS mean_views, avg(v.retention) AS mean_retention
MATCH (v:Video)
RETURN v.title, v.views, v.retention, v.duration, v.published,
       round(100.0 * v.views / mean_views) AS pct_of_mean
ORDER BY v.published DESC LIMIT 40
```

## Recency filter

```cypher
MATCH (t:Thread)
WHERE t.created > datetime() - duration({months: 6})
RETURN t
```

When timestamps are stored as epoch integers instead, `t.created > timestamp() - 15552000000`. Check which one the graph uses — mixing the two returns zero rows without erroring.

## Coverage check

Run before trusting any aggregate. It tells you whether a thin result means "no relationship" or "no data loaded".

```cypher
MATCH (n) RETURN labels(n) AS label, count(*) AS n ORDER BY n DESC;
MATCH ()-[r]->() RETURN type(r) AS rel, count(*) AS n ORDER BY n DESC;
```
