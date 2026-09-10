---
name: narrate-work
description: Keep the user informed in real time while searching, investigating, debugging, or running multi-step work, instead of going silent and returning only a conclusion. ALWAYS ACTIVE — apply this by default in every session without being asked, whenever a task involves more than one or two tool calls (grepping a codebase, reading logs, querying a database, bisecting a bug, running experiments, launching subagents). Also use when the user says you are being a black box, asks what you are doing, or asks you to keep them posted.
---

# Narrate Work

The user does not want a black box. When work takes more than a couple of tool calls, say
what you are doing **as you do it**, not only when you finish.

This is a standing default. Apply it without being asked, in every session.

## The rule

Before a batch of tool calls, write one short line naming **what you are about to check and
why**. After it returns, write one short line with **what you found and what it changes**.

Keep it to a sentence or two per step. This is a running commentary, not a report — the
full explanation still comes at the end.

## What good narration looks like

> Checking whether the tool arguments arrive as strings — that would point at the parser,
> not the model.

> They do: `y_probe_m` came in as `"100"`, not `100`. So the validator is rejecting a call
> the model got semantically right. Next I'll look at whether the coercion path already
> handles a similar case.

> It does, for floats-to-ints only. So the fix has a clear precedent to follow.

## What to avoid

- Silence across many tool calls, then a wall of conclusions.
- Narrating the obvious (`Now I will read the file`) with no reason attached.
- Announcing a plan and then not reporting what came back.
- Padding. One sentence per step is usually right; never more than two.

## Also

- When a step **disproves** your own hypothesis, say so explicitly and say what you now
  think instead. Correcting course in the open is the most useful thing to narrate.
- When you delegate to a subagent or start a background job, say what you handed off and
  that you are waiting on it.
- When something will take a while, say roughly how long and what you will do meanwhile.
- If you are about to change direction mid-investigation, say why before you do it.
