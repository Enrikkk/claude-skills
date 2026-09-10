---
name: drop-jargon
description: Re-explain your own immediately previous message in plain, simple language with no jargon, assuming the user does not already know the technical background. Use whenever the user says they did not understand, that you were too technical, that you were spitting jargon, asks you to explain it simpler or more clearly, or invokes /drop-jargon explicitly.
---

# Drop Jargon

The user did not understand your last message. Explain **the same content again**, from
scratch, in language a smart person with no background in this specific topic would follow.

This is a re-explanation, not a summary and not an apology. The user still wants every
substantive point you made — they just want it in language that actually lands.

## Rules

1. **Re-explain the previous assistant message.** Not the whole conversation, not a new
   topic. If that message covered five findings, the plain version covers all five.
2. **Give the context first, then the point.** The user is not assumed to know the setup.
   Before saying what you found, spend a sentence on what the thing *is* and why anyone
   would look at it. A finding with no context is still incomprehensible, just shorter.
3. **No unexplained terms.** Either use an everyday word, or use the technical word and
   define it inline the first time in the same breath. If a term is not needed, cut it.
   This includes field-specific vocabulary, metric names, file/function/variable names,
   library names, acronyms, and units the user may not read fluently.
4. **Prefer concrete over abstract.** "We slid one signal against the other in time to see
   which shift lined up best" beats "we cross-correlated over lags".
5. **Say what it means for the user**, not just what is true. Each point should land as
   "so this is fine" / "so this is broken" / "so this is a decision you need to make".
6. **Keep the honesty exactly as it was.** Simplifying language must not upgrade a maybe
   into a yes. Keep every hedge, every "not yet verified", every caveat, and every number
   the user's decision depends on. If you were uncertain before, be plainly uncertain now.
7. **Do not lose the numbers.** Keep the ones that matter, but say what each one means and
   whether it is good or bad. A number with no interpretation is jargon too.
8. **Short sentences. Short paragraphs.** One idea per sentence. Break up walls of text.
9. **Do not be condescending.** Simple language, not baby talk. No "as you may know", and
   no over-apologising for the earlier message. One brief orienting line is enough.
10. **No word-swapping.** Do not just trade a hard word for a slightly less hard word.
    Restate the underlying idea in terms of what physically or practically happens.

## Output shape

- Open with one sentence naming what the previous message was actually about.
- Then take the points in the same order as the original message, each with: what the thing
  is, what was done or found, and what it means.
- If the original message ended with open questions, pending work, or a decision the user
  has to make, restate that plainly at the end so nothing gets dropped.
- A little bolding or bulleting helps scanning, but plain prose is fine. Avoid dense
  tables; if the original had a table, explain the rows that carry the point.

## Do not

- Do not introduce new findings, new analysis, or new work in the re-explanation. If you
  realise something new while rewriting, say so separately and mark it clearly as new.
- Do not quietly drop a caveat because it is awkward to phrase simply.
- Do not re-run tools or re-verify. The information is already in the previous message.
