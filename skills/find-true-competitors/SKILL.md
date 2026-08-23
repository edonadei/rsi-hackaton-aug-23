---
name: find-true-competitors
description: Identify who a company actually competes with, from shared-audience and shared-demand evidence in the graph. Use when asked who the competitors are, to prune or validate an existing competitor list, or to assemble a competitive set before keyword or content work.
---

# Find true competitors

Most competitor lists are **category twins**: companies that describe themselves with the same words. They make a tidy slide and a useless strategy, because they may serve a different buyer, at a different price, for a different job.

A **true competitor** is a **substitute** — someone a buyer picks *instead of* the subject when solving the same problem. Substitution is observable in the graph: the same keywords, the same communities, the same buyers, the same head-to-head comparisons. Category resemblance is not evidence. Overlap is.

Read the graph's live schema before querying it. Every label and relationship named
below is a placeholder — substitute what `CALL db.labels()`, `CALL db.relationshipTypes()`
and a sample row actually return.

## 1. Pin the subject

Get three things straight, from the graph and from the user where the graph is silent:

- The **job**: what problem the subject gets hired for, in the buyer's words.
- The **buyer**: who signs off — segment, company size, role.
- The **price shape**: free, self-serve, sales-led, enterprise.

These become the filter in step 3. Skipping them makes every later step subjective. When the graph carries none of it, ask the user rather than inferring from the company's own marketing copy.

## 2. Cast wide

Pull candidates from **every** edge that could indicate substitution, separately, and keep the source of each. Four independent channels beat one deep one:

- **Demand overlap**: entities targeting or ranking for the same keywords.
- **Audience overlap**: entities mentioned in the same communities, threads, or by the same audience nodes.
- **Comparison edges**: explicit "vs", alternative-to, or comparison relationships.
- **Category edges**: shared tags, industries, or topics.

Cast wide here — a candidate wrongly admitted gets cut in step 3, while one never surfaced is invisible for the rest of the analysis. Twenty to forty candidates is a healthy pool.

**Completion criterion**: all four channels queried and their results merged, with each candidate carrying the channels it came from. A candidate found by one channel is a lead; found by three, it is close to settled.

## 3. Cut to substitutes

Score each candidate against the step-1 profile:

| Test | Passing looks like |
| --- | --- |
| Same job | Solves the buyer's problem, whatever the technology |
| Same buyer | Same role and company size signs off |
| Reachable price | Within a band the buyer would actually consider |
| Substitution evidence | Buyer picks one **or** the other, not both |

Sort the pool into three tiers:

- **Direct** — passes all four. The buyer chooses between them.
- **Adjacent** — same buyer, different job. Competes for budget and attention, not for the same purchase.
- **Category twin** — same words, different buyer or different job. Cut, with the reason recorded.

Category twins get **listed with their rejection reason**, never dropped in silence. Half the value of this analysis is telling the user which name they have been benchmarking against for no reason.

## 4. Report

For each Direct and Adjacent competitor:

- Name, tier, and the one-line reason it substitutes.
- **The evidence**: shared keywords, shared communities, comparison mentions — with counts, and the query that produced them.
- Where it beats the subject, and where the subject beats it.
- Overlap strength, so the list is ranked rather than alphabetical.

Then the cut list: every category twin with its rejection reason.

**Completion criterion**: every name in the report carries at least one piece of graph-backed evidence, and every candidate from step 2 appears somewhere — promoted, or cut with a reason. A competitor you cannot evidence goes in a separate "suspected, unevidenced" list, so nobody downstream mistakes a hunch for a finding.

## Where this goes wrong

- **The subject's own marketing** names the competitors it *wants* to be compared to. Weight buyer behaviour over positioning.
- **Big brands dominate raw overlap counts** because they hold more of everything. Normalise — Jaccard over the shared neighbour sets — before ranking.
- **A thin graph** returns a short list that looks decisive. Report the coverage check alongside the list.
