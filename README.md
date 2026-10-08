# how-does-this-scale

An agent skill that makes coding agents **explain how a change behaves as the data behind it grows**, before they call it done.

## Why use it

Features get built and demoed against three rows of mock data. They look fine, pass review, then fall over when a real customer has 50 projects or 200 crons: the collapsible becomes a wall, the dropdown can't be searched, the page fires one request per row, the poll hammers the API.

This skill makes the agent, for every collection the change touches:

1. **Name what grows.** Projects, crons, runs, repos, in the product's own words, traced to where they come from in the code.
2. **Pick the counts.** Today, expected and stretch, with assumptions written down.
3. **Walk through each count.** What the UI looks like (height, search, render cost) and what the data access does (requests per page, N+1, payload size, client vs server filtering, poll frequency, DB scans).
4. **Find where it breaks.** The binding constraint and the count where it stops being acceptable.
5. **Recommend at the right size.** Fine as is, fix now or fix later with a named trigger. No virtualising lists that will never pass 30 rows.

It finishes with a compact table: what grows, where, the counts, the binding constraint, the break point and the verdict.

## Example prompts

- "how does this scale?"
- "what does this collapsible look like with 50 projects?"
- "what happens to this data access pattern when there are 200 crons?"
- "is this N+1? does it need pagination?"

## Install

Clone into your agent's skills directory, for example Claude Code:

```sh
git clone https://github.com/tylers2222/how-does-this-scale-skill ~/.claude/skills/how-does-this-scale
```

Any agent that supports the `SKILL.md` format can load it the same way. The full instructions are in [`SKILL.md`](SKILL.md).
