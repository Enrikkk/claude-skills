---
name: merge-branches
description: Walk the user through resolving git merge conflicts between two branches interactively — one file at a time, one conflict at a time, each decided by an MCQ with a real recommendation. Invoked by the user with /merge-branches <branch-a> <branch-b>, or whenever they ask to merge, reconcile, or integrate a divergent branch (especially a co-founder's or teammate's) and want to review the actual content rather than accept git's or a model's silent auto-resolution. Handles both ordinary content conflicts and add/add conflicts (both sides independently built the same file from scratch since the branches diverged).
---

# Merge Branches

A conflict resolved by silently picking a side (or worse, a model guessing) can quietly delete
a real feature, reintroduce a fixed bug, or fork the definition of a number that's supposed to
be ground truth. This skill treats every conflict as a decision the user makes, informed, one at
a time — never a batch auto-resolve, never "looks fine, taking theirs."

## Plain language, always

Every explanation — inside an MCQ's `question` text, an option's `description`, or a short
chat lead-in — must be understandable to someone with no technical background, the same bar the
`drop-jargon` skill holds itself to: everyday words instead of jargon, a concrete example or
analogy instead of an abstract description, short sentences. "This field never persists because
`saveSettings` only writes two of its three columns" becomes "this box doesn't actually save — it
looks like it works, but the data has nowhere to be stored, so it disappears next time you reload
the page."
Name a technical term only when the user needs it to act (a file path, a function name they'll
see again), and even then explain what it *does* in plain terms alongside it. If the user ever
says an explanation was too technical, that's a real correction — reread `drop-jargon`'s
approach and recalibrate for the rest of the session, not just that one answer.

Default to reading this as `/merge-branches <branch-a> <branch-b>` where `<branch-a>` is the
user's own current work and `<branch-b>` is the branch being merged in (a teammate's, usually).
**If invoked with no args (or only one), do not guess or default to the current branch — ask via
AskUserQuestion.** Run `git branch -a` first so the options are real branch names, not
placeholders; include the current branch and the most plausible "other" branch (most recently
committed, or one clearly named after a teammate) as the first two options, and let the free-text
"Other" cover anything not listed.

## 0. Preconditions — both branches must be real and pushed

Before touching conflicts:

1. `git status` on the user's current branch. If there are uncommitted changes, stop and ask
   how to handle them (commit, stash, or abort) — never start a merge over dirty state.
2. `git fetch --all --prune` so both branches reflect the real remote state.
3. Check whether the user's own branch has unpushed local commits (`git log
   origin/<branch>..<branch>`). If so, **ask before pushing** — pushing publishes work, treat it
   as the outward-facing action it is. Once confirmed, push it. Don't assume silence means yes.
4. Confirm the other branch exists on the remote and note its tip commit and how many commits
   each side has that the other lacks (`git log --oneline A..B | wc -l` and reverse). Report this
   to the user before proceeding — it sets expectations for how big the job is.

## 1. Scope the merge before resolving anything

Do this on a **disposable scratch branch**, never directly on the user's real branch — a bad
merge attempt must be trivially abandonable.

```
git checkout -b _merge_scan_<timestamp-or-suffix> <branch-a>
git merge <branch-b> --no-commit --no-ff
```

Then enumerate:
- `git status --short | grep '^UU\|^AA\|^DD\|^AU\|^UA'` — every conflicted file, and *which kind*
  of conflict (`UU` = both sides modified; `AA` = both sides independently added the same path;
  `DD`/`AU`/`UA` = one side deleted what the other touched — flag these specially, they're easy
  to resolve wrong).
- Conflict-marker count per file (`grep -c '^<<<<<<<' <file>`) — gives a real complexity estimate.
- For every `AA` file, check whether it existed at the merge-base at all
  (`git cat-file -e <merge-base>:<path>`). If it didn't, say so plainly: **both sides built this
  file independently since diverging** — the two versions may implement the same idea
  differently, and diff hunks alone may not be the most meaningful way to compare them. Plan to
  do a whole-file structural comparison for these (exports, main functions, what each computes)
  before diving into hunk-by-hunk resolution.

Report the full scope to the user in one message before resolving anything: file list, conflict
type per file, hunk counts, and which files are add/add. Abort the scratch merge afterward
(`git merge --abort`) — this pass is reconnaissance only, not the real merge.

## 2. The per-file, per-conflict loop

Process **one file at a time**, in an order you choose deliberately (simplest/smallest first is
usually right, so the user builds context before the hardest file — but say why you're ordering
it that way). Do not jump ahead to a later file before the current one is fully resolved and
staged.

Re-run the real merge (not aborted this time) on a fresh scratch branch off the user's current
branch, and work through it file by file.

### For each conflicted file

1. **Analyze every conflict in the file FIRST, silently, before asking anything.** Read the whole
   file (or, if huge, the regions around every conflict marker plus enough surrounding context to
   understand each side's intent). For an add/add file, read both full versions side by side first
   and summarize structurally: what each one exports/does, and where they overlap vs. diverge in
   approach. Use `git log`/`git blame` on each side if it helps establish *why* a piece of code
   exists (e.g. "this null-check was an earlier fix for a null-date bug") — and actually trace
   suspicious-looking additions (does this field get persisted anywhere? does this branch compute
   what it claims to?) rather than taking either side's code at face value. Do this investigation
   work in tool calls, not in chat prose — the user sees the conclusion in the MCQ, not a research
   narration.

   For a genuinely large file (hundreds of conflict lines, or a file big enough that reading both
   full sides would flood the main loop's own context), delegate this analysis step itself to a
   subagent rather than doing it inline — give it the same investigative mandate (trace, don't
   take at face value) and ask it to return a structured, plain-language, MCQ-ready report per
   conflict, including which ones are pure formatting noise vs. real decisions and which ones are
   coupled to each other. This keeps the main loop's context free for running the actual
   conversation instead of holding a 4,000-line file in memory.

2. **Do not post a prose explanation in chat before the questions.** The explanation of what each
   side does, why it likely exists, and your recommendation all belong **inside the MCQ itself** —
   in the `question` text and each option's `description` — not as a message you send first. The
   question text should be self-contained enough that the user could understand and decide the
   conflict from the MCQ alone, without needing to scroll up. A short one-line lead-in per file
   (e.g. "File 3 of 7 — `lib/ledger.ts`, 1 conflict") is fine; a paragraph of standalone analysis
   before the tool call is not — fold it into the question instead.

3. **Ask every conflict in the file as MCQs, back to back, with no narration in between, ALL in
   the same turn — then wait.** Batch up to 4 conflicts per **AskUserQuestion** call (the tool's
   own limit); if a file has more than 4 conflicts, issue consecutive AskUserQuestion calls (4,
   then the next 4, …) in that same turn until every conflict in the file has been asked. Do not
   resolve, edit, or stage anything until you have an answer for every question you asked —
   collect the full set first. Structure each question's options as concrete, mutually exclusive
   choices such as "Keep mine", "Take theirs", "Combine both (as described)", each with a short
   `description` of what actually happens to the code, in plain language (see above). Put your
   recommended option first and label it "(Recommended)" per the tool's convention.
   **Never add a literal "write my own" or "other" option — AskUserQuestion already provides a
   free-text "Other" for any answer that isn't one of the listed choices.** If a conflict is
   trivial (pure whitespace/formatting with no semantic difference — verify this with `diff -w`,
   don't assume from the marker alone), you may resolve it directly without an MCQ, but still say
   so explicitly and briefly rather than resolving it silently.

   If, while investigating, one conflict turns out to need more than an MCQ can carry (the answer
   genuinely depends on something you couldn't have known before asking — e.g. a chosen option
   turns out to require touching other files), it's fine to ask a focused follow-up. Keep it to
   the same plain-language bar, and if other questions from the same file are still unanswered,
   send the follow-up alongside them in one batch rather than trickling questions out one at a time.

4. **Once every question for the file is answered, hand the actual coding to a subagent — the
   main loop itself never writes or edits code.** This is a hard rule, not a judgment call: even a
   one-line conflict resolution goes through the Agent tool. The main loop's job is analysis,
   questions, and verification of what comes back; a subagent's job is every Write/Edit. Launch
   one Agent call per file (or several in parallel when a file's decisions split into clearly
   independent pieces) with the exact decision for every conflict spelled out ("Conflict 1: keep
   mine because X; conflict 2: combine both — keep theirs' Y but preserve ours' Z; …") and precise
   instructions to remove every marker and produce valid, working code — never leave a marker in
   place or paste both sides concatenated. If a decision requires real implementation work beyond
   the conflicted lines themselves (a new DB column and its migration, wiring a field through a
   backend function, updating a downstream consumer in another file, extending a verification
   harness), fold that into the same subagent instructions — it's still "the coding for this
   decision," not a separate escalation. Always require the subagent to report back precisely what
   it changed (files touched, line-level summary, the full resolved content of each changed
   region) so the result can be checked, not just trusted.

   **Don't block on the subagent — keep the conversation moving.** Agent calls run in the
   background and notify on completion; while one file's subagent is working, move on to
   analyzing and asking the MCQs for the *next* conflicted file rather than sitting idle waiting
   for a report. This is what actually makes the loop fast: the user is never stuck watching code
   get written, and you're never stuck waiting on a subagent when there's more to ask about. Only
   pause the conversation once every file has either been asked about or is blocked on a prior
   file's answer (e.g. a later file depends on how an earlier add/add file's shared logic was
   resolved).

   **Going quiet while a subagent runs is a failure mode, not a neutral default — treat it as
   something to actively avoid.** Before saying some version of "waiting for the subagent," check
   whether there is truly nothing else to do: another file not yet asked about, a follow-up worth
   raising, a prior resolution worth re-verifying now that more context exists. Only when
   genuinely everything is blocked on the one in-flight subagent is it acceptable to wait — and
   even then, use the time productively (re-check an earlier file's staged diff, re-run a
   cheap verification, prepare the closing summary) rather than posting a bare "waiting" message
   and stopping. A single background subagent should essentially never be a reason the whole
   turn goes idle.

5. **After every conflict in the file is resolved**, show the user the resulting file (or the
   changed regions if it's large) so they can sanity-check the assembled whole — a set of locally
   correct choices can still combine into something structurally broken (duplicate function
   definitions, an import removed by one hunk but still used by code kept from the other).

6. **Stage the file** (`git add <file>`) only once the user confirms it looks right. If the
   project has a fast per-file check (e.g. a typecheck or linter that can target one file), run it
   now and surface real errors before moving on — catching a break here is cheaper than catching
   it at the end.

7. Move to the next conflicted file and repeat. Keep a running short summary as you go (e.g. "3 of
   7 files done") so the user can see progress in a long session.

## 3. Closing out

Once every conflicted file is staged and no `git status` conflict markers remain:

1. Run the project's real verification steps if discoverable (typecheck, build, test suite —
   check `CLAUDE.md`/`package.json` scripts for what "done" means in this repo). Report results
   plainly, including failures — don't declare success before verifying.
2. Show a final summary: what was merged, the key decisions made (one line each), and anything
   left ambiguous or deferred.
3. Ask before committing the merge commit, and ask separately before pushing or before fast-
   forwarding/replacing the user's real branch with the scratch branch's result — these are each
   outward-facing or hard-to-reverse and deserve their own confirmation, not one blanket yes.
4. Clean up the scratch branch only after the user confirms the result is safely where they want
   it (merged into their real branch, or pushed) — don't delete the only copy of resolved work
   prematurely.

## What this skill is not

Not a tool for silently auto-resolving conflicts "the sensible way," and not a one-shot batch
job. If asked to "just merge it," push back gently and explain this skill exists specifically so
every conflict gets seen and decided by the user — that's the point, not overhead to skip.
