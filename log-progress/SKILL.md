---
name: log-progress
description: Write an end-of-session/end-of-task progress record so a future agent starts with full context. Invoked by the user with /log-progress. Logs to BOTH the project memory directory and the project's own state doc. Records what was done, what was decided and why, the numbers that were measured, the traps discovered, and — importantly — what was left as future work.
---

# Log Progress

The user compacts often and hands work to fresh agents. This skill writes the record that
makes that survivable: everything a competent stranger would need to pick this up cold.

Write to **both** destinations, every time:

1. **Project memory** — `~/.claude/projects/<project>/memory/`, following the memory rules
   already in the system prompt (one fact per file, frontmatter with `name` / `description` /
   `metadata.type`, plus a one-line pointer in `MEMORY.md`). Update the file that already
   covers the topic rather than creating a near-duplicate.
2. **The project's own state doc** — the markdown the repo already uses for this
   (a `STATE.md`, a `project_*.md`, a `README`'s state section). If the repo has
   no such doc, say so and put everything in memory rather than inventing a new file.

Memory survives across repos and sessions; the repo doc travels with the code and shows up
in a diff. Neither alone is enough.

## Log only — do not turn this into an audit

This skill **writes down what you already know**. It does not kick off a fresh
investigation, re-verify deployments, or run `git status` to discover facts. If something
is genuinely unknown, record it as unknown. Running new checks here buries the log under
tool output and is not what was asked for.

The one thing you must always include beyond "what happened" is **future work** (below).

## What actually helps a future agent

**Decisions, with the reason and the rejected alternatives.** "Picked model A" is
nearly useless. "Picked model A; rejected model B because the runtime ships no tool
parser for it, and model C because it doesn't fit in available memory" prevents the next
agent from re-running the whole comparison.

**Measured numbers, verbatim.** `0.031 s/step`, `9.56 GiB`, `24/24 checks passed`. Numbers
are the part that cannot be re-derived from the code, and they are what makes a claim
checkable later.

**Traps and gotchas**, especially ones that cost time. "Editing the repo under `~/app-copy`
changes nothing — the running service imports from a different installed location."
"A bare script doesn't read config.json until `_load_config()` runs." These are the
highest-value lines in the whole log.

**Where things live** — exact paths, service names, commands, URLs, run ids. Anything that
took a search to find should never need finding twice.

**Corrections to earlier beliefs.** If something recorded previously turned out wrong, say
so plainly and update or delete it. A confidently wrong memory is worse than none.

## Future work — always include this

Close every log with what is *not* done, so the next agent has a direction rather than only
a history. Cover:

- **Explicitly deferred** — things decided against *for now*, with the condition that would
  change the answer ("revisit if X").
- **Blocked** — what is stuck and on what, including anything the user was asked to do.
- **Known-incomplete** — parts knowingly left partial, and what "finished" would mean.
- **Natural next step** — if the work has an obvious continuation, name it.

Mark clearly whether each item is *agreed with the user* or *your own suggestion*. A future
agent must not mistake your idea for the user's instruction.

If nothing is outstanding, say that explicitly — "no open threads" is real information.

## Writing it

- Lead with current state, not chronology. A future agent needs "where things stand" first;
  the story of how you got there is context, not the headline.
- Be specific enough to act on and short enough to read. Prefer a table of numbers to a
  paragraph describing them.
- Flag what is verified versus assumed, and say how it was verified.
- Don't log what the repo already records — code structure, diffs, commit history. Log the
  things that live only in this conversation: why, what was ruled out, what broke.
- Don't log conversation trivia, praise, or a blow-by-blow of your own tool calls.

Finish by telling the user, in two or three lines, what you recorded and where.
