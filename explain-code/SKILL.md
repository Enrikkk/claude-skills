---
name: explain-code
description: Explain a function, class, or file block-by-block in plain English. Use whenever the user asks to explain, walk through, break down, or understand what a piece of code does (e.g. "explain this function", "walk me through X", "what does this file do"), or when invoked explicitly as /explain-code <path or symbol>.
---

# Explain Code

Produce a block-by-block plain-English walkthrough of the requested function, class, or
file, aimed at a confident developer who hasn't read this specific code yet but knows how
to program. The goal is understanding, not a summary — the user should come away knowing
what each part does AND why it's structured that way.

## Process

1. **Read the target in full** before writing anything. Don't explain from a partial read.
2. **Follow imports/calls as needed.** If a block calls a helper, a config field, or another
   module in a way that's load-bearing for understanding it (not just incidental), go read
   that too and fold it into the explanation. Don't treat every call as a black box, but
   don't go chasing unrelated code either — follow only what's needed to explain the block
   in front of you.
3. **If something is ambiguous or its rationale isn't evident from the code/comments/docstrings**,
   say so plainly rather than guessing ("no documented reason found for X") — do not invent
   a plausible-sounding justification.

## Output format

1. Start with a **one- or two-sentence summary** of what the function/file does overall and,
   if relevant, its role in the bigger system (e.g. "this is the one code path used for both
   training and inference").
2. Then walk through the code **block by block, in source order**. For each block:
   - Quote the relevant snippet (or reference it by line range for long blocks).
   - Give a plain-English explanation of what it does and, where it matters, *why* — the
     invariant it protects, the bug it avoids, the physical/domain reason behind a design
     choice — not just a restatement of the syntax.
3. Close with anything that ties the blocks together (e.g. how state flows from one block
   to the next) if that isn't already obvious from the walkthrough.

## Style rules

- Plain English over jargon; define any domain term the first time it's used.
- Don't pad with generic software-engineering commentary ("this is a good practice
  because...") — stay specific to this code.
- Don't create files or artifacts for this — reply directly in the conversation unless the
  user asks for a saved doc.
- Keep code quotes short and targeted; don't repeat the entire file back to the user.
