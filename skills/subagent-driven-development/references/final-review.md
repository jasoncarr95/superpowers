# Final Review and Finish

Part of superpowers:subagent-driven-development. Every task in the plan is
complete.

The final whole-branch review gets a package too: run
`bash scripts/review-package PLAN_FILE MERGE_BASE HEAD` (MERGE_BASE = the commit the
branch started from, e.g. `git merge-base main HEAD`) and include the
printed path in the final review dispatch, so the final reviewer reads
one file instead of re-deriving the branch diff with git commands. Dispatch
on the most capable available model (see SKILL.md's Model Selection), using
superpowers:requesting-code-review's
[code-reviewer.md](../../requesting-code-review/code-reviewer.md). Point it at
the ledger's deferred-minor and parked lines so it can triage which must be
fixed before merge.

### Cross-cutting sweeps

This review is the only one that can see the whole branch, so give it the
questions only a whole-branch view can answer. A defect whose _cause_ sits in a
file this branch changed and whose _symptom_ sits in a file no task touched is
invisible to every scoped reviewer by construction — each one saw a correct
diff and approved it.

Derive a short list of sweep questions from the branch diff and hand them to
the final reviewer as named checks. Common generators:

- **A new table, column, or entity** → which existing delete, archive, cascade,
  or cleanup paths must learn about it? Does this project actually _enforce_
  the constraint it declares, or is the declaration decorative?
- **A decision revised late** (see the pre-flight scan) → grep the branch for
  prose, comments, or commit messages still asserting the superseded form.
- **A new parameter on a shared function** → is every call site updated, or do
  some silently take a default?
- **A new export** → who consumes it? Is it dead, or duplicating something?
- **A new effect, cache, or piece of state in a shared component** → does an
  existing effect need the new dependency?

Two real defects escaped eleven scoped reviews on one branch this way: a
declared `ON DELETE CASCADE` that the runtime never enforced, so four
untouched delete paths silently stranded rows; and a superseded design claim
that had spread to six files. Both were one sweep question away.

### The fix wave

If the final whole-branch review returns findings, dispatch ONE fix subagent
with the complete findings list — not one fixer per finding.
Per-finding fixers each rebuild context and re-run suites; a real
session's final-review fix wave cost more than all its tasks combined.
Then run exactly one scoped re-review of the fix wave
(`bash scripts/review-package PLAN_FILE FIX_BASE HEAD` over the fix range,
[re-review-prompt.md](../re-review-prompt.md)).
Adjudicate any residual findings as in the fix loop's breaker
(references/fix-loop.md): park with rulings, or rule on the load-bearing
ones and ledger what you decided. Only the four things that stop a running
plan (top of SKILL.md) stop you here. There is no second fix wave —
residual load-bearing findings surface to your human partner when
finishing-a-development-branch presents the options.

### Finish

#### 1. Harvest the ledger — before archiving anything

Before you archive anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, breaker adjudications, all of them — into
your final message under "Rulings I made", in the order you made them, each
with what it costs if wrong. The list is exhaustive: if the ledger holds a
ruling, the list holds it. That list is the only place the decisions you
took on your human partner's behalf reach them — they read it and rework
whatever you got wrong. A ruling that only survives in the archive was a
decision made in secret.

Git holds the code and the fixes. It does **not** hold the findings you chose
not to fix: deferred minors, parked findings with their rulings, accepted
deviations, plan defects you ruled on, and anything marked UNREVIEWED. Those
exist only in a git-ignored file about to leave the live path for an archive
the project never reads. Archiving the workspace without harvesting is the
silent discard this skill forbids everywhere else.

Read the ledger and pull every line that is a _finding_ rather than
bookkeeping — `minor (deferred)`, `parked`, `DEVIATION`, `PLAN DEFECT`,
`UNREVIEWED`, `BLOCKED`, and any controller ruling. Then write them somewhere
committed:

- Find the project's own convention first. Its agent-context file
  (`AGENTS.md`, `CLAUDE.md`) or its notes tree usually names a backlog, an
  ideas file, or a code-review-report lifecycle. Use it — a harvest filed
  where the project already looks is one the project will actually read.
- Absent a convention, ask your human partner where these should live rather
  than inventing a file they will never open.
- Group by what a reader would act on (user-visible, correctness, tests,
  cosmetic), keep each item to a line or two with enough location to act on,
  and say where they came from.
- Commit the harvest.

Findings already fixed on the branch need no harvest — the diff and the
commit messages carry those.

#### 2. Archive the workspace

Only once the final whole-branch review is clean, its fixes are merged, and
the harvest is committed, move the workspace out of the live path:

```bash
ROOT=$(git rev-parse --show-toplevel)
WS=$(bash scripts/sdd-workspace PLAN_FILE)   # the path setup printed
mkdir -p "$ROOT/.superpowers/sdd/done"
mv "$WS" "$ROOT/.superpowers/sdd/done/$(basename "$WS")-$(date +%F)"
```

Archive the path `sdd-workspace` prints, never one rebuilt from the plan's
basename: a plan whose filename another plan already owns gets
`<slug>-<parent>[-N]`, and the bare basename would name that other plan's
live workspace. If the destination already exists, add a `-2`, `-3` suffix
rather than merging into it. The move is what marks the plan finished: a
workspace at the live path means an interrupted run to resume, and a directory under
`done/` is the run's record — ledger, briefs, reports, review packages —
kept for later process audits and skill work. `done/` is git-ignored by
the default `*` in `.superpowers/sdd/.gitignore`; if the repo committed its
own `.gitignore` there, check `git status` after the move. The
workspace is moved, never deleted. Sibling directories belong to other
plans; leave them alone.

#### 3. Hand off

Use superpowers:finishing-a-development-branch. Carry any UNREVIEWED ledger
entry into that conversation as a named caveat — it is your human partner's
call whether to merge work no second pair of eyes has seen.

| Excuse | Reality |
|--------|---------|
| "Every task review passed, so the branch is clean" | Scoped reviewers cannot see a defect whose symptom lives in a file no task touched. That is what the cross-cutting sweeps are for. |
| "The git history is the record — the workspace can go" | Git has the code and the fixes. It never had the findings you chose _not_ to fix, nor the record of how the run went. Harvest first, then archive to `done/`. |
| "The final review already triaged the minors" | Triage picks which to fix. The ones it declines still need a committed home — a ledger sitting in `done/` that nobody opens is a discard with extra steps. |
| "Nobody will read the archive — just `rm -rf` it" | The archive is the only record of how the run went: rulings, fix rounds, review packages. Git-ignored files cost nothing to keep; a deleted run's audit trail is unrecoverable. |
