---
name: optimize-keyword-strategy
description: Turn the graph's keyword data into a ranked, clustered plan of what to target next. Use when asked which keywords to go after, to find gaps against competitors, to prioritize an existing keyword list, or to plan content around a topic.
---

# Optimize keyword strategy

A keyword list sorted by search volume is a wish list. The high-volume terms are held by sites with a decade of authority, and chasing them buys nothing.

Strategy means picking the **winnable**: terms where demand is real, intent matches what the subject sells, and the incumbent is beatable. Winnability is the ranking axis; volume is one input to it.

Invoke `dynamic-query-neo4j` for schema recon. Competitor terms come from `find-true-competitors` — a gap against a category twin is not a gap.

## 1. Map the current footprint

Everything the subject already ranks for or targets, with position, volume, and the page that holds it.

Look first for **near-misses**: positions 4–20. A term at 8 is often two internal links and a rewrite away from 3, while a brand-new term is months of work. The cheapest wins in this whole analysis live here, and a strategy that opens with net-new terms has skipped them.

## 2. Find the gaps

Terms the competitive set holds and the subject does not, using the gap query in `dynamic-query-neo4j`'s patterns. Rank by **how many rivals cover it**: a term every rival holds is structural demand, while a term one rival holds may be their idiosyncrasy.

## 3. Classify intent

Sort every candidate — near-miss and gap alike — by what the searcher wants:

- **Transactional** — ready to buy. "pricing", "buy", "vs", "alternative", brand + product.
- **Commercial** — comparing. "best X", "X for Y", "X tools".
- **Informational** — learning. "how to", "what is", "guide".
- **Navigational** — looking for a specific brand.

Intent decides the page you build and the conversion you can expect. A perfect informational ranking sells nothing directly, which is fine when the strategy says so and a waste when nobody noticed.

Drop terms whose intent the subject cannot serve, and say which you dropped. Ranking for a query you have no answer to earns a bounce.

## 4. Score winnability

For each candidate:

```
winnability = demand x intent_fit x (1 - incumbent_strength) x existing_asset_bonus
```

- **demand** — search volume, damped. Treat this as log-scale; the gap between 100 and 1,000 matters more than 10,000 to 11,000.
- **intent_fit** — how directly the term maps to what the subject sells (0–1).
- **incumbent_strength** — difficulty score, or the authority of whoever holds the top three.
- **existing_asset_bonus** — multiply for near-misses and for terms an existing page already half-covers.

Use whatever difficulty and volume properties the graph carries. When it carries none, say so and rank on rival coverage and intent alone — an honest two-factor ranking beats a fabricated score.

## 5. Cluster before recommending

Group the survivors by the **question they ask**, not by string similarity. Ten phrasings of one question are one page, and shipping ten pages for them splits the subject's own authority ten ways.

Each cluster gets: a head term, its members, total volume, dominant intent, and the single asset that should own it.

**Completion criterion**: every recommended term sits in exactly one cluster, and every cluster names the one page that will target it.

## 6. Report

A ranked table, highest winnability first:

| Cluster | Head term | Volume | Intent | Difficulty | Current position | Action | Effort |
| --- | --- | --- | --- | --- | --- | --- | --- |

`Action` is one of: **optimize** an existing page, **create** a new one, or **consolidate** several thin ones. Split the table into *this quarter* (near-misses, low difficulty) and *the long game* (structural gaps worth building toward).

Close with what you deliberately left out and why — the high-volume terms that are unwinnable, and the ones whose intent the subject cannot serve. That list stops someone re-proposing them next month.
