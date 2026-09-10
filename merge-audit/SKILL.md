---
name: merge-audit
description: Deep, after-the-fact audit of a merge that has already happened — whether it was done through /merge-branches or manually — hunting for code that got silently skipped, dropped, or broken during the merge. Invoked by the user with /merge-audit [<branch-a> <branch-b>] [<merge-commit>], or whenever they ask to double-check, verify, sanity-check, or audit a merge they just did or did earlier. With no arguments, infers the branches from the most recent merge in this conversation; if that isn't clear, asks. Produces a structured findings report and a persistent log file under merge_logs/, then — for any real findings — walks the user through resolving each one via an MCQ, the same interactive pattern /merge-branches uses, before dispatching fixes to a subagent. Static analysis only (diffs, reference integrity, typecheck, the project's test suite) — it does not boot the app or exercise code live; that's /test-features' and /run's job.
---

# Merge Audit

`/merge-branches` gets every conflict decided by a human, one at a time — but a merge can still go
wrong in ways that never raise a conflict marker at all: a modify/delete where the moved-to file
silently never received the deleting side's real change, an auto-merge that combines two
non-overlapping hunks into something that no longer holds together, a stray unrelated edit that
rides along into the commit, a reference to something the other side renamed or deleted. None of
that trips a git conflict. This skill's whole job is finding that class of problem — the merge that
"succeeded" but is still wrong — after the fact, whether the merge was done through
`/merge-branches`, a plain `git merge`, or by hand.

**This is a verification pass, not a second merge-branches run.** It does not re-litigate decisions
that were already made and recorded — it checks that the code actually reflects them, and hunts for
everything that was never a decision at all because nobody noticed it needed one.

## Plain language, always

Same bar as `/merge-branches` and `drop-jargon`: every finding, every MCQ, every recommendation —
in the question text, an option's `description`, or a chat lead-in — has to be understandable with
no technical background. "The modify/delete on `db/ledger.ts` never got its safeQuery wrapper ported
to `schema.ts`" becomes "A teammate's fix for a database error was thrown away when the file got
reorganized — nobody re-applied it in the new location." Name a file or function only when the user
needs it to act, and explain what it *does* in plain terms alongside it.

## 0. Identify what to audit

Default invocation is `/merge-audit` with no arguments — figure out the merge from context, per
this fallback chain, stopping at the first one that gives a clean, unambiguous answer:

1. **Explicit arguments given** (`/merge-audit <branch-a> <branch-b>`, or a merge commit SHA) — use
   them directly, no inference needed.
2. **The current conversation clearly just performed or discussed one specific merge** (branches
   named, a merge commit identified, `/merge-branches` was just run) — use that merge directly and
   say so in one line before starting ("Auditing the feature-branch ← teammate-branch merge,
   commit abc1234, from earlier in this conversation."). Reuse whatever decision record already
   exists in the conversation (what was decided per conflict, and why) — it's the single best source
   for phase 3's "does the code match the recorded intent" check, better than reconstructing intent
   from the diff alone.
3. **Not clear from conversation** — look at the repo itself: `git log --merges -1 --format="%H %P %s"`
   on the current branch for the most recent real merge commit. If exactly one plausible candidate
   turns up, propose it and ask for confirmation (don't silently assume — the user's own words were
   "ask if it is at least a little bit unclear," and an inferred git-log guess is exactly that kind
   of unclear) before proceeding.
4. **Still unclear** (multiple recent merges, no merge commit findable, or a manual/squash merge that
   left no 2-parent commit at all) — ask directly via `AskUserQuestion`, with real branch names from
   `git branch -a` as options (same pattern `/merge-branches` uses for its own no-args case). If no
   real merge commit exists at all, also ask for the target branch's commit **right before** the
   merge happened — some of the checks below need that reference point and can't reconstruct it on
   their own from two branch tips alone (the target's current tip usually already contains the
   source branch as an ancestor, which makes a naive `git merge-base` useless for finding what
   *should* have changed).

Once identified, resolve exactly:
- **Target** — the branch/commit holding the "already merged" result (what's being audited).
- **Source(s)** — the branch(es) that were supposed to be merged in.
- **Pre-merge target** — the target's tip immediately before the merge (the merge commit's first
  parent when one exists).
- **Merge-base** — `git merge-base <pre-merge-target> <source>`.

Report this back in one line before moving on, so the user can catch a wrong guess immediately
rather than after a long audit runs against the wrong thing.

## 1. Reconstruct the real scope — don't trust a raw tip-to-tip diff

The most common way an audit misleads itself: diffing the two branches' *current tips* against each
other. If either branch has any history of its own beyond the merge point, that diff is mostly noise
— unrelated commits neither branch's merge was ever about. Always diff **each source branch against
the merge-base**, not against the other branch's tip:

```
git diff --name-only <merge-base> <source>       # source's REAL changes
git diff --name-only <merge-base> <pre-merge-target>   # target's REAL changes (the other side)
```

This is the true scope: every file either side actually touched. Cross-reference it against the
merge commit's own conflict list if one exists (`git show <merge-commit>` shows the merge diff;
combined with the two diffs above you can tell which files were auto-merged vs. flagged conflicts —
useful context but not the scope boundary itself).

If this conversation already has a merge-branches session's scope report in it, reuse it rather than
re-deriving — just re-verify it's still accurate against the actual final commit (people sometimes
amend a resolution after the fact).

## 2. Per-file verification — the core of the audit

For every file in the real scope (step 1), classify it and verify the specific thing that class of
change is most likely to have gotten wrong:

- **Only the source changed this file, target didn't** → confirm the target's current version
  actually contains the source's real change (not just *a* version of the file — the actual delta).
  If it's missing, that's a finding — either it was silently dropped, or it was a deliberate decision
  that never got applied.
- **Only the target changed this file, source didn't** → nothing to verify here on its own, but note
  it for step 3's reference-integrity pass (something else in the source might reference what
  changed here).
- **Both sides changed it, non-overlapping regions (git auto-merged, no conflict raised)** — the
  sneaky case. A clean auto-merge is not proof of correctness: check that BOTH sides' real hunks
  actually landed in the final file, and specifically look for cross-hunk semantic breakage — one
  side renamed or removed something the other side's untouched-but-nearby code still relies on, in a
  way that doesn't show up as a text conflict because the two edits never touched the same lines.
- **Both sides changed it, git flagged a conflict** — confirm the resolution is a real, coherent
  combination of both sides' intent, not an accidental full pick of one side that silently discards
  real logic from the other. Cross-check against any recorded decision (conversation record, commit
  message, ADR) — does the code actually do what was decided, not just something plausible-looking?
- **Add/add** (both sides independently created the same path since diverging) — confirm whichever
  version was kept was chosen after an actual structural comparison (what each one exports/does), not
  just "the one that happened to end up in the tree."
- **Modify/delete** (one side deleted or moved the file, the other side kept editing the old path) —
  **the single highest-value check in this whole skill.** If the file was deleted because it moved
  or got restructured (not because the feature was actually removed), confirm the deleting side's
  final tree contains an equivalent of the OTHER side's real change, applied at wherever the file
  actually ended up — not just that the old path is gone. This is exactly the class of bug that
  produces "a feature quietly stopped working after the merge" with no error, no conflict, nothing in
  `git status` — it looks clean and isn't.
- **Rename/rename, delete/delete, or any other conflict class git surfaces** — same instinct: confirm
  the final state reflects a real decision about BOTH sides' intent, not an artifact of however git's
  auto-resolution or a rushed manual fix happened to land.

For a file large enough that reading both full sides would flood context, delegate that single
file's analysis to a subagent (same reasoning as `/merge-branches`' own large-file delegation) — but
the main loop must independently spot-check at least the highest-risk claims a subagent reports
before trusting them into a finding. A false "all clear" from an unverified subagent is worse than
no check at all, because it's the thing this whole skill exists to prevent people from relying on.

## 3. Cross-cutting integrity checks

Beyond the file-by-file pass, check these across the whole scope:

- **Reference integrity.** Anything either side renamed, moved, or removed — a function, an export, a
  type, a database column/table, a config key, an environment variable — grep the whole scope (and
  reasonable adjacent code, not just the changed files) for lingering references to the old name.
  This is where "modify/delete" bugs and "auto-merge" bugs both tend to actually manifest as broken
  code, not just missing code.
- **Scope drift.** Any file that changed in the target and is NOT explained by either side's real
  diff from the merge-base (step 1's scope) is a red flag — an unrelated, possibly-accidental change
  that rode along. (This is a real thing that happens, not a hypothetical — it happened during the
  session this skill's own design conversation grew out of: an unrelated, broken-looking edit sitting
  in the working tree that had nothing to do with the merge.) Flag it; don't assume it's fine just
  because it's unrelated to anything conflict-worthy.
- **Leftover conflict markers.** `grep -rn "^<<<<<<<\|^=======$\|^>>>>>>>"` across tracked files only
  — explicitly exclude `node_modules/`, `.venv/`, `dist/`, `build/`, and any other vendored/dependency
  directory, since their own source can contain those exact character sequences legitimately (false
  positives that waste a look if not filtered up front).
- **Dependency manifests.** If either side touched a lockfile or package manifest, confirm the
  lockfile is actually consistent with the manifest post-merge (an orphaned entry, a version mismatch)
  — this project's own deploy checklist has hit exactly this class of bug before.
- **Build integrity.** Re-run the project's real verification commands fresh — don't trust that
  "typecheck passed earlier in the session" still holds after every file's resolution landed. Use
  whatever the project defines as its full check (its README/CLAUDE.md's verify section); if genuinely
  undiscoverable, at least run whatever build/typecheck/test tooling the repo's own package
  manifest / CI config implies.

**Out of scope, deliberately:** booting the app, clicking through a UI, or exercising a changed code
path live. That's real, valuable testing — it's just a different skill's job (`/test-features` for a
feature-level pass, `/run` to just get the app up). Say so explicitly if the user seems to expect
this skill to do it, rather than silently skipping it.

## 4. Verify candidate findings before they count

Anything found in steps 2–3 is a **candidate**, not a finding, until it survives an independent
re-check — reread the actual current code at the cited location and confirm the problem is real, not
an artifact of a stale diff view or a misunderstanding of what the code does elsewhere. This mirrors
`/code-review`'s own verify pass: a finding that doesn't survive re-verification is dropped
silently, not reported as a caveat-laden maybe. Peace of mind depends on the report being trustworthy,
not exhaustive — a false alarm here costs more than a missed one, because it teaches the user to
stop trusting the audit.

## 5. Write the persistent log, then report

1. **Log file first.** Create `docs/merge_logs/` if the repo already has a `docs/` directory at its
   root (matching an existing living-docs convention), otherwise `merge_logs/` at the repo root.
   Write one file per audit run: `YYYY-MM-DD_<source>-into-<target>.md` (append `_2`, `_3`, ... if the
   same pair is audited more than once the same day). Contents: what was audited (branches, merge
   commit, merge-base), what was checked (a short recap of steps 1–3 so the log is self-contained),
   and every surviving finding from step 4 — even if the list is empty, still write the log; a clean
   result is itself the durable record ("audited on this date, N files / M checks, nothing found").
   Each finding gets a `Resolution: PENDING` line, to be filled in during step 6.
2. **Report in chat via `ReportFindings`**, ranked most severe first, using the verified list from
   step 4 (empty array if nothing survived — that's a valid, good outcome, not a failure to find
   something). Follow it with a short plain-language recap of what was actually checked (file counts,
   which integrity checks ran, build/test result) — the reassurance of "here's everything I looked
   at" matters as much as the findings list itself for genuine peace of mind.

## 6. Resolve findings — the merge-branches pattern, applied to problems instead of conflicts

If there are zero findings, stop here — report the clean result, the log file's location, and you're
done.

If there are real findings, walk through them the same way `/merge-branches` walks through
conflicts:

1. **Batch up to 4 findings per `AskUserQuestion` call**, back to back in one turn, no narration in
   between — collect every answer before changing anything. Put the explanation and your
   recommendation **inside the question** (question text + option `description`), not as chat prose
   beforehand. Real, mutually exclusive options only (e.g. "Fix it now", "Not actually a problem —
   mark resolved", "Leave it for later, just record it") with your recommended option first and
   labeled "(Recommended)". Never add a literal "handle it myself" option — `AskUserQuestion`'s own
   free-text "Other" already covers that.
2. **Every code change goes through a subagent** — the main loop never calls `Write`/`Edit` itself,
   even for a one-line fix, exactly like `/merge-branches`. Give the subagent the precise decision for
   each finding it's fixing and require it to report back exactly what changed.
3. **Verify what comes back** — re-run the relevant check (typecheck, the specific test, a targeted
   re-read) before accepting a fix as done, not just trusting the subagent's own report of success.
4. **Update both records** once resolutions are settled: the log file's `Resolution:` lines (what was
   decided and what actually happened), and a second `ReportFindings` call with `outcome` set per
   finding (`fixed` / `skipped` / `no_change_needed`) — this is exactly the tool's documented
   re-report pattern for "after fixes were applied."
5. Don't go idle waiting on a subagent if there's another finding's questions still to ask, or an
   earlier one's fix to re-verify — same anti-idle discipline as `/merge-branches`.

## What this skill is not

Not a second `/merge-branches` pass — it doesn't re-open settled decisions, only checks that the
code actually reflects them and hunts for what nobody ever got the chance to decide because it never
raised a conflict. Not a live/behavioral test — no booting the app, no clicking through screens; that
belongs to `/test-features` or `/run`. And not a silent auto-fixer — every real finding gets a
decision from the user before any code changes, the same principle `/merge-branches` is built on.
