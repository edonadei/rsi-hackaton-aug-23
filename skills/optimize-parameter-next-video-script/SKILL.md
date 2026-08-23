---
name: optimize-parameter-next-video-script
description: Set the parameters of the next video script — topic, title, hook, length, structure, CTA — from past video performance in the graph. Use when planning the next video, choosing a title or hook, diagnosing an underperforming video, or drafting a script outline.
---

# Optimize parameters: next video script

A script is a bundle of **parameters**, each one a knob with a value: topic, title pattern, hook, length, pacing, structure, CTA placement. Past videos already carry evidence about which values work for *this* channel — evidence that generic advice ("hook in 3 seconds!") cannot see.

The job: read what the channel's own data says, hold the settled parameters constant, and change **one** deliberately.

Invoke `dynamic-query-neo4j` for schema recon.

## 1. Pull performance with a baseline

Every past video with whatever metrics the graph carries: views, retention (average view duration, or 30-second hold), click-through rate, watch time, publish date, duration, title, topic.

Compute the **channel baseline** first — mean and median per metric. A video with 40% retention is a hit or a flop depending entirely on whether the channel's norm is 25% or 55%, and a raw number without its baseline is unreadable.

Normalise for age: a video from last week has not finished accumulating views. Compare like-aged videos, or use a rate.

**Completion criterion**: every video carries its metrics *and* its ratio to the baseline.

## 2. Isolate what moves

Group the videos by each parameter and compare against the baseline:

| Parameter | Split by |
| --- | --- |
| Topic | Subject cluster |
| Title pattern | Question / number / how-to / claim; length; whether it names a specific tool |
| Length | Bucketed — under 5 min, 5–12, 12–25, over 25 |
| Hook type | Question, promise, cold open, story, result-first |
| Structure | List, tutorial, narrative, teardown |
| CTA placement | Start, middle, end, none |
| Publish timing | Day and hour |

Report **effect sizes with sample counts**, not rankings: "how-to titles average 1.4x baseline views (n=8)". A parameter value with two videos behind it is an anecdote and must be labelled one.

Two traps live here, and both produce confident nonsense:

- **Confounding.** The long videos may also be the tutorials. When one parameter's groups line up with another's, say the two are tangled instead of picking a winner.
- **Reading noise.** Under ~5 videos in a group, no difference is real. Report the direction, mark it unconfirmed, and move on.

**Completion criterion**: every parameter above is either given a data-backed reading or explicitly marked "insufficient data" — no silent omissions.

## 3. Set the parameters

Produce a parameter sheet for the next script. Each row: the value, and the evidence behind it.

| Parameter | Setting | Evidence |
| --- | --- | --- |
| Topic | | |
| Title | 2–3 options | |
| Length | | |
| Hook (first 15s) | | |
| Structure | | |
| CTA + placement | | |
| Thumbnail concept | | |

Where the data settles a parameter, **hold it**. Where the data is silent, take the channel's current convention and mark the row unevidenced.

Then pick exactly **one** parameter to change as an experiment, with a written prediction and the metric that will judge it. One change per video is what makes the result readable; two changes make the next analysis unattributable, which is how a channel accumulates data it can never learn from.

## 4. Draft the outline

Write the script outline to the parameter sheet:

- **Hook** — the first 15 seconds, written out verbatim. This is the parameter with the largest measured effect on nearly every channel, and it is too short to leave as a description.
- **Beats** — each section with its target duration, summing to the chosen length.
- **Retention saves** — where past videos lost viewers, and what to put there.
- **CTA** — the exact wording, at the placement the data supports.

**Completion criterion**: every parameter on the sheet appears in the outline, the beat durations sum to the target length, and the experiment is stated with its prediction and its metric.

## Report

Parameter sheet, then outline, then the experiment. Close with what the data could not answer and how many more videos it would take to answer it — that turns the gap into a plan instead of a blind spot.
