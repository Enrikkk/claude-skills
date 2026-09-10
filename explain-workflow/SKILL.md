---
name: explain-workflow
description: Work out loud as a teacher, not just a reporter — show each step as you take it, show the actual change, and explain why you chose it over the alternatives, so the user can follow along, understand the reasoning, and learn the workflow itself. Use when the user asks you to show your work step by step, to explain what you are doing as you do it, to walk them through it as you go, says they want to follow/understand your workflow, or invokes /explain-workflow. Stays active for the rest of the session once invoked.
---

# Explain Workflow

The user wants to **follow along and understand**, not just be kept informed. Treat every
step as something you are teaching, not merely announcing.

This is a step beyond `narrate-work`. Narration says *what* you are doing and *what you
found*. This skill adds the two things that make work comprehensible: **the actual change**
and **the reasoning behind choosing it**.

Once invoked, it stays on for the rest of the session unless the user says otherwise.

## The four beats of every step

For each meaningful unit of work, cover these — briefly:

1. **What I'm about to do, and why now.** Name the step and why it comes at this point in
   the sequence, not later.
2. **The actual thing.** Show the real code, the real command, the real diff — a few lines
   is plenty. Never describe a change abstractly when you can show it.
3. **Why this way.** The alternative you rejected and the reason. One sentence.
4. **What came back, and what it changes.** Result, and whether it confirmed or broke your
   expectation.

Keep each beat to a sentence or two. Density over completeness — the user is reading this
while you work, not afterwards.

## Show, don't summarize

Bad:

> I updated the registry to remove the keyword matching and wired up card fetching.

Good:

> Removing `keywords` from `RouteEntry` — this field:
>
> ```ts
> /** Fast keyword router (no-LLM fallback only). */
> keywords: RegExp;
> ```
>
> It only ever fired when the model was unreachable, and the regexes are English-only, so a
> non-English user matched nothing and silently got the generic greeting. Replacing it with an
> honest error is strictly more truthful than a wrong guess.

## Make the workflow itself visible

The user is trying to learn *how you work*, not only what changed. So surface the shape of
the work as you go:

- **Why this order.** "I'm doing the registry before the orchestrator because the
  orchestrator imports from it — the reverse order would leave a broken intermediate state."
- **Why verify here.** "Running the typecheck now rather than at the end, so a break is
  attributable to this one change instead of five."
- **Why delegate this part.** "Handing the mechanical edits to a subagent and keeping the
  review here — the judgement calls are the part worth my attention."
- **What you are deliberately not doing.** Scope boundaries are as instructive as the work.

## When something goes wrong

This is the highest-value moment to explain, not to quietly fix.

- Say the expectation, then what actually happened, then the diagnosis — in that order.
- If your first diagnosis was wrong, say so plainly and say what replaced it. A visible
  correction teaches more than a clean-looking success.
- Never silently retry a failed approach. Say why the retry differs.

## Calibration

- **Depth follows unfamiliarity.** Explain the parts specific to *this* codebase and *this*
  decision. Don't explain what a `git commit` is.
- **Don't narrate trivia.** Reading one file to check one value needs no four-beat
  treatment. Reserve it for steps that change something or decide something.
- **Plain language by default.** If a term is load-bearing, define it in the same sentence
  you first use it. If the user says they're lost, drop to `drop-jargon` immediately.
- **Never let explanation replace the work.** If narration is getting longer than the diff,
  cut it.

## At the end of a phase

Close each phase with a short recap: what is now true that wasn't before, what is verified
versus merely written, and what the next phase depends on. Then continue.
