# The Fix Loop

Part of superpowers:subagent-driven-development. The task review reported
spec ❌, a Critical or Important finding, or a ⚠️ item you confirmed as a real
gap.

Before the loop starts, two routes leave it immediately:

- Record Minor findings in the progress ledger as you go
  (`Task <N>: minor (deferred): <one-liner>`), and point the final
  whole-branch review at that list so it can triage which must be fixed
  before merge. A roll-up nobody reads is a silent discard. Minor findings
  never enter the loop. Write each one to be readable months later by someone
  without this session: name the file, and say what is wrong rather than that
  something is. Triage decides which get fixed; Finish's harvest step is what
  keeps the rest — the ledger itself is git-ignored and ends up in an
  archive nobody reads by default.
- A finding labeled plan-mandated — or any finding that conflicts with
  what the plan's text requires — is yours to rule on: weigh the finding
  against the plan text, decide with the spec as the binding authority, and
  ledger the ruling before you act on it. Do not dismiss the finding because
  the plan mandates it, and do not dispatch a fix that contradicts the plan
  without a recorded ruling.
Everything else enters the loop. A fix round is one fix dispatch plus one
scoped re-review. Five rounds maximum per task:

**Rounds 1-3 — resume the original implementer.** Send it the open findings
verbatim. Its context is intact: it knows the task, the code, and its own
choices. If your harness cannot send another message to a live subagent,
dispatch a fresh implementer carrying the brief path, the report-file path,
and the findings — the report file is the persistent memory either way.

**Rounds 4-5 — dispatch a fresh implementer on a more capable model** (per
SKILL.md's Model Selection), with the brief path, the report-file path, the open
findings, and this framing: "A prior implementer attempted this task
[N] times; you own it now. Read the report file for what was tried." A loop
that survives three resumes usually means the implementer cannot see its
own problem — fresh eyes and a capability bump in one move.

**Every round, either way:** the implementer fixes, re-runs the tests
covering the amended code, appends its fix report to the same report file,
and returns the short contract. Before re-dispatching the reviewer, confirm
the fix report contains the covering tests, the command run, and the
output; when the fix adds or changes a guard or regression test, also
confirm its mutation evidence (guard removed → test fails → restored →
green) is in the fix report. Dispatch the re-review once everything
required is present. Name the covering test files in the fix message — a one-line fix does not need the
whole suite.

**The re-review is scoped.** Run `bash scripts/review-package PLAN_FILE FIX_BASE HEAD`
where FIX_BASE is the head the previous review saw, and dispatch
[re-review-prompt.md](../re-review-prompt.md) with the findings list, the
brief, the report file, and the printed diff path. The re-reviewer verdicts
each finding ADDRESSED or NOT ADDRESSED and flags new breakage in the fix
diff only. New Critical/Important breakage in the fix diff joins the open
findings list. Out-of-scope observations go to the ledger as deferred
minors — they never extend the loop.

**After each round,** append to the ledger:
`Task <N>: fix round <R>/5 (<X> addressed, <Y> open — <finding one-liners>; commits <a7>..<b7>)`

Never fix findings yourself in the controller session — your context stays
clean for coordination, and controller fixes skip review.

**When dispatch is unavailable.** A spend limit, quota, or outage can make
subagent dispatch impossible mid-loop. The rule above then has no compliant
path, so it yields — but only under a disclosure contract. First confirm the
tree: a killed subagent may have left partial edits, so check `git status` and
`git log` before touching anything. Then, if you fix directly, all four of:

1. Name the fix in the commit body as controller-applied and unreviewed.
2. Append `Task <N>: UNREVIEWED — <what you fixed> — reason: <why dispatch
failed>` to the ledger.
3. Verify harder than a reviewer would, because no one else will: run the
   covering tests, and for any regression test guarding a _silent_ failure,
   prove it discriminates — remove the fix, confirm the test fails, restore it,
   confirm no residue.
4. Surface it as a named caveat at finish, not a footnote. Your human partner
   is now the review.

Waiting for the budget to reset is also valid. What is never valid is fixing
directly and reporting the result as reviewed.

**The breaker.** When round 5's re-review still leaves findings open, stop
dispatching. Adjudicate each open finding yourself — you hold the plan and
the cross-task context the reviewer lacks:

- **The reviewer is wrong, or the point is contestable:** park it —
  `Task <N>: parked — <finding> — Ruling: <why the code stands>`. The final
  review sees both sides.
- **Real, but nothing downstream builds on it:** park it the same way, with
  a ruling that says it's real and deferred.
- **Real and load-bearing** — a later task builds on it, or it reveals a
  plan defect: rule on the smallest change that unblocks the dependent work,
  ledger it as `Task <N>: Ruling: <finding> — <what you decided and why>`,
  and carry it into the next task's dispatch. Parking a structural failure
  silently lets every dependent task build on it. Stop only when the defect
  leaves every path forward a guess.

Adjudicate only at the cap. Adjudicating earlier to end a loop is
pre-judging with a different name. Every adjudication is a ledger entry —
a silent discard is forbidden.

| Excuse | Reality |
|--------|---------|
| "Dispatch is down, so I'll just fix it and move on" | You may fix it. You may not call it reviewed. Commit body, ledger `UNREVIEWED` line, discriminating-test proof, and a named caveat at finish — all four. |
