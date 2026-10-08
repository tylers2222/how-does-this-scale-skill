---
name: how-does-this-scale
description: Make the agent explain how a change behaves as the data behind it grows, before calling it done. For every list, table, collapsible, dropdown, query, fetch, loop, poll or job in the change, name the thing that grows (projects, crons, runs, repos, users), pick realistic counts now and later, and walk through what the UI and the data access do at each count, where it breaks and what the fix is. Use when the user asks "how does this scale", "what does this look like with 50 projects", "what happens when there are 200 crons", "will this hold up", "is this N+1", "does this need pagination or virtualisation", or before shipping any feature that renders or fetches a collection, even if they don't say "scale".
---

# How does this scale

Most features are built and demoed against three rows of mock data. They look fine, pass review, and then fall over when a real customer has 50 projects or 200 crons: the collapsible becomes a wall, the dropdown can't be searched, the page fires one request per row, the poll hammers the API, the query scans a table on every render.

This skill makes you think that through out loud, against the actual code, before the change is called done.

**Rule: for every collection the change touches, you can say how many there will be, what the code does at that count, and at what count it stops being acceptable.**

## 1. Find the things that grow

Read the change (the current diff, the named files, or the plan) and list every place that handles a collection:

- **UI:** lists, tables, collapsibles and accordions, tabs, dropdowns and selects, sidebars, charts, cards per item, filter chips.
- **Data access:** fetches, query hooks, API handlers, DB queries, loops that call out per item, polling and refetch intervals, subscriptions, caches, cron or job fan-out.

For each, name the **noun that grows** in the product's own words: projects, applications, repositories, crons, runs, deployments, members. Trace it to where it comes from (which endpoint, which table, which prop) with `file:symbol`. Don't guess the source. Search for it.

Skip things that are genuinely bounded by design (an enum of five statuses, the days of the week). Say that they're bounded and why, in one line.

## 2. Pick the counts

For each noun, pick three counts:

- **Today:** what the mock data or current customers have.
- **Expected:** what a healthy, real customer has in a year.
- **Stretch:** the largest plausible customer, roughly 10x expected.

Use numbers from the user, the code, seed data, docs or tickets when they exist. When they don't, estimate and write the assumption next to the number ("assuming ~4 crons per application and 50 applications → 200 crons"). Counts multiply across nesting: 50 projects × 10 applications × 4 crons is 2,000 rows if the page shows them all.

## 3. Walk through each count

For each collection, say concretely what happens at each count. Be specific to the code, not generic.

**For UI, ask:**
- How tall is it? 200 rows at 40px is 8,000px. Does the user still find what they came for?
- Is there search, filtering, sorting, grouping or pagination? Does the default state (expanded or collapsed, first page) still make sense?
- Does each item render something expensive (a chart, a nested query, a timer)? Multiply that cost by the count.
- Does it still fit at phone width, and in the narrowest container it lives in?
- What do the empty, one-item and loading states look like? Scale includes the small end.

**For data access, ask:**
- How many requests does one page load make? Is any of it N+1 (one request or query per item)?
- How big is the payload at the stretch count? Is the whole collection fetched to show ten items?
- Is filtering, sorting or pagination done on the client after fetching everything, or on the server?
- What does the query do in the database: is the filter column indexed, is it a full scan, does it join per row?
- How often does it run? A poll every 5 seconds × open tabs × users. A cron that fans out one job per item.
- What happens when one item is slow or failing: does it block the rest?

## 4. Find where it breaks

For each collection, name the **binding constraint**: the one thing that actually breaks first in the user's range. It's often not the obvious one. Request count, payload size, scroll length, render cost, DB scan, rate limits, or simply "a human can't scan 200 rows" are all valid answers.

Then name the **break point**: the count where it stops being acceptable, and how you worked it out.

## 5. Recommend, at the right size

For each collection, pick one:

- **Fine as is.** It holds past the stretch count. Say why. This is a good answer, not a failure to find a problem.
- **Fix now.** It breaks below the expected count. Give the concrete change (server-side pagination, a batched endpoint, search in the select, collapse by default, virtualise the list, move the filter into the query, add an index, back off the poll).
- **Fix later.** It breaks between expected and stretch. Name the trigger that should prompt the fix ("when any org has more than 100 crons") so it isn't forgotten.

Don't over-engineer. Virtualising a list that will never pass 30 rows is cost with no benefit. Match the fix to the break point.

Reuse what the codebase already has. Before recommending pagination, search for an existing paginated list, a shared table, a search-select or a batched query hook, and point to it.

## Output

Lead with a compact table, then the detail only where something breaks:

| What grows | Where | Today / expected / stretch | Binding constraint | Breaks at | Verdict |
|---|---|---|---|---|---|
| crons per org | `CronList` → `GET /crons` | 10 / 200 / 2,000 | one request per cron for status | ~50 | Fix now |
| projects in scope picker | `ScopePicker` | 5 / 50 / 500 | no search, can't scan the list | ~30 | Fix now |
| statuses | `cronStatusMeta` | 4 (bounded) | n/a | n/a | Fine as is |

Under the table, for each **Fix now** and **Fix later** row: what happens at the break point in one or two sentences, and the recommended change with the `file:symbol` it touches or reuses.

End with every assumption you made about counts, so the user can correct them. If the user corrects a count, redo the affected rows.

## When you're also writing the code

If you're building the feature rather than reviewing it, run this before you call the work done, and fix the **Fix now** rows as part of the change. Check the result against mock data at the expected count, not just the three-row seed: add or generate enough rows to see what a real customer would see.
