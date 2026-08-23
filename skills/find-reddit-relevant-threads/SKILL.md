---
name: find-reddit-relevant-threads
description: Find Reddit threads worth entering for a product, topic, or keyword cluster, each with a fit reason and an angle. Use when asked where to engage on Reddit, to mine communities for audience language and objections, or to source community distribution for a launch.
---

# Find Reddit-relevant threads

Keyword-matching Reddit produces a list nobody can act on: threads from 2019, threads in subs that ban promotion, threads where the asker already picked a tool. Every one of those costs a person twenty minutes to open and discard.

A thread is **worth entering** on four conditions at once, and failing any one disqualifies it:

1. **Fit** — the asker's problem is one the subject genuinely solves.
2. **Alive** — recent enough, or still drawing traffic, that a reply gets read.
3. **Open** — the subreddit's rules permit what you intend to post.
4. **Answerable** — there is something useful to say beyond naming the product.

Read the graph's live schema before querying it; the labels below are placeholders.

## 1. Build the query terms

Start from the subject's **problem language**, not its product language. People post "spent all day reconciling spreadsheets", never "seeking reconciliation automation platform".

Derive terms from three sources:

- The keywords the subject targets, especially informational and commercial intent.
- Competitor names — "alternative to X" threads are the highest-intent threads on Reddit.
- Symptom phrases: what the problem feels like before the buyer knows a category exists.

## 2. Search the graph

Query indexed threads and comments across all three term sets, keeping subreddit, score, comment count, timestamp, and the matched term for every hit.

Search **wide** and filter in step 3. Reddit's vocabulary drifts far from marketing vocabulary, so a narrow query returns a clean, tiny, unrepresentative set.

**Completion criterion**: all three term sets queried, results merged and deduplicated by thread ID.

## 3. Score each thread

| Signal | Strong | Weak |
| --- | --- | --- |
| Fit | Asker states the exact problem the subject solves | Topic adjacency only |
| Recency | Under 6 months, or still gaining comments | Over 2 years and quiet |
| Engagement | Comments and score above the sub's norm | Zero comments |
| Openness | Sub permits tool mentions in context | Sub bans self-promotion |
| Room | No accepted answer, or answers are thin | Asker already chose and thanked |

Drop anything failing Fit or Openness outright — the first wastes the reader's time, the second gets the account banned and the brand named in a callout thread.

Old threads with steady search traffic ("evergreen") stay in, flagged as such: a reply there is read by searchers for years, even with the asker long gone.

## 4. Report

Ranked, strongest first. Per thread:

- **Link and title**, subreddit, age, score, comment count.
- **Why it fits** — quote the asker's own words, one line.
- **The angle** — the specific useful thing to say, written so someone can act on it without reopening the thread.
- **Mention rule** — whether naming the product is appropriate here, and what the sub's rules require.

Group by subreddit rather than by score. A person works one community at a time, and the grouping matches how the work actually gets done.

**Completion criterion**: every listed thread has a link, a quoted fit reason, and a written angle. A thread you cannot write an angle for fails the *answerable* test — cut it.

## Participation rules

These are not optional polish. Reddit punishes violations at the account and brand level, and the punishment lands publicly.

- **Disclose affiliation** whenever mentioning the subject. "I work on X" costs nothing and is the difference between a helpful comment and a ban.
- **Answer the question first.** The reply should be useful even with the product mention removed.
- **Read the sidebar** before posting. Rules vary per sub and several ban vendor participation entirely.
- **One thread, one reply.** A pattern of the same pitch across many threads reads as spam because it is spam.

Surface these rules in the output — the person acting on the list needs them, and they are the part most often skipped.
