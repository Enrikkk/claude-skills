---
name: log-progress-compact
description: Log all session progress AND write a resume handoff immediately before the user compacts the context. Invoked by the user with /log-progress-compact. Does everything /log-progress does, then records the in-flight working state — the task mid-execution, the exact next action, what is already verified, and what is settled — so that after the compaction you continue seamlessly, as if it never happened.
---

# Log Progress (pre-compaction)

The user is about to compact. Everything not written down now is gone. This skill has two
jobs, in order:

1. **Everything `/log-progress` does** — follow that skill in full: write to both the
   project memory directory and the project's own state doc; record decisions with their
   reasons and rejected alternatives, the measured numbers, the traps, where things live,
   and the future work. Do not shortcut it because the context is full; this is the last
   chance to record any of it.

2. **A RESUME block** — the part that is unique to compaction, below.

Same constraint as `/log-progress`: **write down what you already know.** Do not open a new
investigation or re-verify things at the moment you are trying to preserve context.

## The RESUME block

Put it at the **top** of the memory file and the state doc, clearly marked (e.g.
`🎯 RESUME HERE`), because a fresh reader must hit it before the history. It is addressed to
*you, three minutes from now, with no memory of this conversation*. Write it in the
imperative.

Cover, in this order:

**The one-line situation.** What is being worked on and how far along it is.

**The immediate next action.** Not the goal — the literal next step, concrete enough to act
on without re-deriving it. "Run X, then check Y" beats "continue the migration".

**In-flight state.** The things that silently break a resumed session:
- edits made but **not** deployed/built/applied, and where they need to go
- background tasks or long-running jobs still going, and how to check them
- anything half-applied that would be dangerous to leave (say so loudly)
- files opened or being edited when the session ended

**Already verified — do not redo.** List what has been tested and how, so the resumed
session doesn't burn its fresh context re-running passing checks. Re-verifying settled work
reads as dawdling, not rigor.

**Settled decisions — do not re-litigate.** Choices the user already made, with one line of
reason each. After a compaction the temptation is to reopen these; naming them prevents it.

**Hard-won facts.** Anything expensive to rediscover: exact paths, ids, service names,
commands with their real arguments, measured numbers, the shape of a fix. If it took a
search or an experiment to learn, it goes here.

**Open questions for the user**, if any are pending — with the options, so the thread can be
picked up without re-explaining it.

## What to avoid

- A summary of the conversation. This is a handoff, not minutes. Ordering matters: state
  first, next action second, history last or not at all.
- Vagueness that reads fine now and is useless later. "Fix the config issue" — which file,
  which key, which value?
- Recording only successes. What failed, and what was already ruled out, is what stops the
  resumed session repeating it.
- Burning the remaining context on tool calls. Write the log; don't go re-derive it.

## Finish

Tell the user, in two or three lines, what you recorded and where, and confirm you are ready
for the compaction. If something genuinely cannot be preserved in writing — a decision only
they can make, an action only they can take — say so now, plainly, while they can still act
on it.
