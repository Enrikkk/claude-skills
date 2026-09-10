# claude-skills

A collection of [Claude Code](https://claude.com/claude-code) skills — reusable, invokable
workflows for Claude Code that live under `~/.claude/skills/<name>/SKILL.md`.

To use one, copy its folder into your own `~/.claude/skills/` (user-level, works across all your
projects) or a project's `.claude/skills/` (project-scoped). Claude Code picks up any `SKILL.md`
it finds there automatically, and each can be invoked explicitly with `/<name>` or triggered
implicitly when your request matches its description.

## Skills in this repo

| Skill | What it does |
|---|---|
| [`narrate-work`](narrate-work) | Always-on: narrates multi-step work in real time (what's being checked and why, what was found) instead of going quiet and returning only a final answer. |
| [`explain-workflow`](explain-workflow) | Works out loud as a teacher for the rest of the session — shows each step as it's taken and why, so you can follow and learn the workflow itself, not just get the result. |
| [`explain-code`](explain-code) | Explains a function, class, or file block-by-block in plain English. |
| [`drop-jargon`](drop-jargon) | Re-explains Claude's own previous message in plain language, assuming no technical background. |
| [`merge-branches`](merge-branches) | Walks you through resolving git merge conflicts interactively — one conflict at a time, each decided by a multiple-choice question with a real recommendation, instead of silent auto-resolution. |
| [`merge-audit`](merge-audit) | Deep audit of a merge *after* it's already happened — hunts for code silently skipped, dropped, or broken by the merge that never raised a conflict marker in the first place. |
| [`test-features`](test-features) | Runs a deep testing pass on a just-implemented feature: functional correctness, edge cases, data integrity, the project's own test suite, and live/real-environment behavior. |
| [`log-progress`](log-progress) | Writes an end-of-session progress record (what was done, what was decided and why, what's left) so a future agent — or a future you — starts with full context. |
| [`log-progress-compact`](log-progress-compact) | Same as `log-progress`, plus a resume handoff written immediately before compacting context, so work continues seamlessly afterward. |

## Why skills

A skill packages instructions Claude Code loads on demand — a checklist, a workflow, a house
style — so you don't have to re-explain the same process every session. See Anthropic's own
[Claude Code docs](https://docs.claude.com/en/docs/claude-code) for how skills, slash commands,
and `CLAUDE.md` project instructions fit together.

## License

MIT — use, copy, and adapt freely.
