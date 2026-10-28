---
name: mcq-explain
description: Resolve a task, an issue, or a whole set of open issues through multiple-choice questions (the built-in AskUserQuestion tool, or any equivalent MCQ tool) where EVERY question carries its own deep, plain-language explanation of the issue — context, the problem, a concrete example, what each answer leads to, cost and risk — so the user learns the issue and decides it in the same step ("parallel explanation-resolution"). Invoked with /mcq-explain [issues or scope], or whenever the user asks to go through issues/questions/decisions "in an MCQ", "with MCQs", "explain it in a question so I can answer there", or says the previous options were not explained well enough. With no argument, the scope is every open issue, question, and pending decision raised in the conversation so far. Once invoked, keep using it for every user decision for the rest of the session unless the user says to stop.
---

# MCQ-Explain

The user wants to steer complex work by answering questions, not by reading walls of text and then
writing instructions from scratch. Each MCQ is therefore two things at once: **a short lesson** that
makes the issue fully understandable, and **a decision point** whose options (or the free-text
"Other") tell you exactly how to proceed. Done well, the user can resolve ten tangled issues in a few
rounds, understanding every one. Done badly, they answer "explain it better" and the round is wasted.

The bar is simple: **someone who only sees the question dialog — nothing you wrote before it — must
understand the issue deeply enough to decide it well.**

## 1. Build the issue ledger first

Before asking anything:

1. **Collect every open item** in scope: questions the user asked you, questions you asked them that
   are still unanswered, decisions you proposed but nobody settled, blockers, and anything an earlier
   answer explicitly deferred ("let's discuss later"). With no argument, scan the whole conversation;
   with an argument, take exactly that scope.
2. **Split compound items into atomic decisions.** One question = one decision. "Should we add the
   baseline and how?" is two decisions if the "how" only matters after a "yes".
3. **Find the dependencies.** If the options of B depend on the answer to A, B goes in a later round.
   Independent decisions go in the same round.
4. **Separate decisions from facts and actions.** Something you can verify yourself (a file, a
   number, a paper, a config value) is not a question for the user — go find it. Something only the
   user can *do* (log in, approve a payment, restart a VPN) is not an MCQ either — state it plainly
   in one line, outside the questions.
5. Keep the ledger in your head (or a scratch file for long sessions): each item is `open`,
   `asked`, `resolved → decision`, or `needs re-explanation`.

Tell the user the plan in **one or two lines** before the first call: how many decisions, in how many
rounds, and anything that is blocked on them. No more prose than that.

## 2. Do the homework before you explain

An explanation is a set of claims. The user will decide on the strength of them, so they must be true.

- **Ground every claim** that matters to the decision: read the code, check the numbers, look up
  the paper. If a claim is a guess, say "I expect" or "likely", and never present it as known.
- **Check your own premises especially hard** when an option you recommend depends on them. A
  confident "this costs nothing and works" that turns out wrong costs more trust than the whole
  question was worth.
- **When the issue rests on outside knowledge** (how others did it, whether a method is standard),
  research it first and cite the source inside the question in a few words ("as in Gu et al. 2022,
  the S4 authors").
- **Get the numbers that make the options comparable**: time, compute, money, how many files,
  how reversible.

## 3. Anatomy of a question

Write the `question` text as a small self-contained lesson, in this order. Use the parts the issue
needs. A simple choice needs two sentences; a hard research or design decision may need 200–350
words, and that is fine.

1. **HEADLINE IN CAPS** — the decision in a few words ("SLIDING-WINDOW BASELINE, HOW TO ADD IT.").
2. **Where this comes from** — the context needed to care: what we found, what was measured, what
   is blocked. Real numbers with their meaning ("ppl 32.6, the worst of the six ~75M models").
3. **The problem in plain words** — what is unknown or broken and why it matters to the user's goal.
4. **A concrete example or analogy** — the single most effective device. A tiny worked case
   ("in A B C D E F with K=3, at D it must answer A") beats a definition every time.
5. **What each outcome means** — if relevant, what we will learn or what changes downstream for each
   answer ("if it wins at 16K, the limit is X; if it fails, it is Y").
6. **The precise ask** — end with the question itself.

**Self-contained means:** never write "as I said above", "the option from before", or a term defined
only in chat. The dialog may hide everything else. Re-state what is needed, briefly.

**Plain language means** the `drop-jargon` bar: everyday words, short sentences, every technical
term defined in passing the first time it appears in that question ("a fade-out — the kernel's
built-in weakening of far-away tokens"). Name a technical identifier only when the user will meet it
again, and say what it does.

## 4. Options that actually decide

- **2–4 options, mutually exclusive, each a real path.** No filler options that only reword another.
- **Each `description` says what will happen if chosen**: the concrete next action, its cost (time,
  compute, money), what it gives up, and its main risk. One to three sentences.
- **Recommend when you have a view:** put that option first and append " (Recommended)" to its
  label, and make the description say *why* in one clause. If you genuinely have no view, say so in
  the question text instead of faking one.
- **Labels are 1–5 words**, distinguishable at a glance. `header` is a ≤12-character chip.
- **Don't add "Other" or "Explain more"** — the tool adds free text automatically. Add a
  "Discuss first" or "Decide later" option only when deferring is a real, sensible path.
- **Use `preview`** when the user must compare concrete artifacts (code snippets, layouts, configs,
  sample outputs); it renders side by side.
- **Use `multiSelect`** only when the choices genuinely combine (e.g. "which of these checks to run").

## 5. Batching and rounds

- **Up to 4 questions per call** (the tool's limit). For more independent decisions in the same round,
  issue consecutive calls in the same turn without narration between them.
- **Order within a round:** the decision everything else hangs on first; then the most consequential;
  quick, low-stakes ones last.
- **Do not act on a round until every question in it is answered.**
- **Between rounds**, a one-line bridge is enough ("Round 2 of 3 — depends on your answer about X.").

## 6. Reading the answers — the part that decides whether this works

Read every answer, especially free text, for what it actually says. Free text often:

- **Asks for a deeper explanation** ("I still don't get it", "explain further"). Then *re-explain
  differently*, never by repeating the same text louder:
  change the angle (a new example, a step-by-step walk-through, an analogy from the user's own field);
  name the specific concept that was likely missing; say what the thing is NOT, if confusion is
  likely; go one level more basic. Then ask the decision again in a new MCQ.
- **Challenges a premise** ("didn't we already test this?", "but X works differently"). Treat it as
  a real point. Check it. If the user is right, say so plainly and adapt. If partly right, say which
  part holds and which doesn't, with the reason, inside the next question.
- **Asks you to verify or research something** before deciding. Do it, report what you found
  (including where you were wrong), and only then re-ask.
- **Changes their mind or the scope.** Follow the newest answer. Update the ledger.
- **Adds a new requirement** in passing ("also do it for the other models"). Add it to the ledger.

Never treat a clarification request as a decision, and never proceed on a question the user didn't
actually answer. If an answer is ambiguous, ask a short follow-up MCQ rather than guessing.

## 7. Closing the loop

When every item in scope is resolved:

1. **Summarise the decisions** in a compact table: issue → decision → what happens next. Note
   anything still blocked and on whom.
2. **Record them** where the project keeps state (its progress log, memory files) if it has one, with
   the reasons, so a future session doesn't re-ask settled questions.
3. **Act on them**, unless an action is outward-facing or hard to reverse (publishing, pushing,
   deleting, spending money), which still needs its own explicit go-ahead if the decision didn't
   already give one.

## 8. Mistakes to avoid

- A paragraph of analysis in chat before the call — it belongs inside the questions.
- Two decisions crammed into one question, or one decision split across two.
- Asking the user something you could have looked up.
- An unverified claim inside an explanation, stated as fact.
- Options without costs or consequences, so the user can't tell them apart.
- Repeating the same explanation after "I don't understand".
- Re-asking a decision the user already settled (check the ledger and project notes first).
- Stopping after one round while ledger items are still open.

## 9. A good question vs a weak one

Weak: *"Should we add a caching layer? Options: Yes / No."*

Good: *"CACHING FOR THE REPORTS PAGE. The reports page takes ~9 s to load because it re-runs the same
three database queries on every visit, even though the numbers only change once a night. A cache
means keeping the last answer in memory and reusing it, like writing a phone number on a sticky note
instead of looking it up every time. The trade-off is staleness: if the nightly job runs late, people
see yesterday's numbers until the cache refreshes. What should we do?"* — Options: **"Cache until the
nightly job (Recommended)"**: load time drops to under 1 s; numbers are never staler than the latest
nightly run; ~half a day of work. **"Cache for 1 hour"**: simpler, but numbers can lag by up to an
hour after the nightly job. **"No cache, speed up the queries"**: always fresh; ~2–3 days of work,
gains uncertain.

## No MCQ tool available

If the agent has no AskUserQuestion-style tool, emulate it: numbered questions, each with the same
explanation anatomy, lettered options with the same descriptions, the recommended option first, and
an explicit "or tell me in your own words". Then stop and wait for the answers.
