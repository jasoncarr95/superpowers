---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development

Execute plan by dispatching a fresh implementer subagent per task, a task review (spec compliance + code quality) after each, and a broad whole-branch review at the end.

**Why subagents:** You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Core principle:** Fresh subagent per task + task review (spec + quality) + broad final review = high quality, fast iteration

**Narration:** between tool calls, narrate at most one short line — the
ledger and the tool results carry the record.

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping. The only reasons to stop are the four named below, or all tasks complete. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the plan, so execute it.

**Rulings, not stalls.** A running plan does not wait on a human. Conflicts,
ambiguities, plan defects, a cap you would have asked to exceed — decide
them. The spec is the binding authority, the plan is its argument, and your
judgment settles what neither answers. Record every decision in the ledger as
`Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep
going. A wrong ruling costs rework your human partner can see and undo; a
session parked on a question costs their whole day and buys nothing.

Four things stop you, and only these: an irreversible or destructive
operation; a security-sensitive action; a side effect outside this worktree
that norms say you ask about first (a merge, a push to a shared branch, a
publish); and a plan so broken that every path forward is a guess. For those,
stop and ask.

## When to Use

```dot
digraph when_to_use {
    "Have implementation plan?" [shape=diamond];
    "Tasks mostly independent?" [shape=diamond];
    "Stay in this session?" [shape=diamond];
    "subagent-driven-development" [shape=box];
    "executing-plans" [shape=box];
    "Manual execution or brainstorm first" [shape=box];

    "Have implementation plan?" -> "Tasks mostly independent?" [label="yes"];
    "Have implementation plan?" -> "Manual execution or brainstorm first" [label="no"];
    "Tasks mostly independent?" -> "Stay in this session?" [label="yes"];
    "Tasks mostly independent?" -> "Manual execution or brainstorm first" [label="no - tightly coupled"];
    "Stay in this session?" -> "subagent-driven-development" [label="yes"];
    "Stay in this session?" -> "executing-plans" [label="no - parallel session"];
}
```

**vs. Executing Plans (parallel session):**

- Same session (no context switch)
- Fresh subagent per task (no context pollution)
- Review after each task (spec compliance + code quality), broad review at the end
- Faster iteration (no human-in-loop between tasks)

## The Process

```dot
digraph process {
    rankdir=TB;

    subgraph cluster_per_task {
        label="Per Task";
        "Dispatch implementer subagent (./implementer-prompt.md)" [shape=box];
        "Implementer asks questions?" [shape=diamond];
        "Answer questions, provide context" [shape=box];
        "Implementer implements, tests, commits, self-reviews" [shape=box];
        "Generate review package, dispatch task reviewer (./task-reviewer-prompt.md)" [shape=box];
        "Spec ✅ and quality approved?" [shape=diamond];
        "Finding conflicts with plan text?" [shape=diamond];
        "Rule on the conflict, ledger the ruling" [shape=box];
        "Fix round R of 5: R≤3 resume implementer; R≥4 fresh implementer, more capable model" [shape=box];
        "Dispatch scoped re-review (./re-review-prompt.md)" [shape=box];
        "All findings addressed?" [shape=diamond];
        "R = 5?" [shape=diamond];
        "Adjudicate each open finding" [shape=box];
        "Any load-bearing finding?" [shape=diamond];
        "Rule and continue; stop only if every path forward is a guess" [shape=box];
        "Park findings in ledger with rulings" [shape=box];
        "Update plan checkboxes in task commit,\nresolve final HEAD,\nappend completion to ledger,\nmark todo complete" [shape=box];
    }

    "Setup: worktree, ledger check, read plan, pre-flight review" [shape=box];
    "More tasks remain?" [shape=diamond];
    "Dispatch final code reviewer (../requesting-code-review/code-reviewer.md)" [shape=box];
    "Final findings? ONE fix dispatch, one scoped re-review, adjudicate residuals" [shape=box];
    "Final review clean: harvest ledger findings to the project backlog, commit, then delete this plan's workspace" [shape=box];
    "Use superpowers:finishing-a-development-branch" [shape=box style=filled fillcolor=lightgreen];

    "Setup: worktree, ledger check, read plan, pre-flight review" -> "Dispatch implementer subagent (./implementer-prompt.md)";
    "Dispatch implementer subagent (./implementer-prompt.md)" -> "Implementer asks questions?";
    "Implementer asks questions?" -> "Answer questions, provide context" [label="yes"];
    "Answer questions, provide context" -> "Implementer implements, tests, commits, self-reviews";
    "Implementer asks questions?" -> "Implementer implements, tests, commits, self-reviews" [label="no"];
    "Implementer implements, tests, commits, self-reviews" -> "Generate review package, dispatch task reviewer (./task-reviewer-prompt.md)";
    "Generate review package, dispatch task reviewer (./task-reviewer-prompt.md)" -> "Spec ✅ and quality approved?";
    "Spec ✅ and quality approved?" -> "Update plan checkboxes in task commit,\nresolve final HEAD,\nappend completion to ledger,\nmark todo complete" [label="yes"];
    "Spec ✅ and quality approved?" -> "Finding conflicts with plan text?" [label="no"];
    "Finding conflicts with plan text?" -> "Rule on the conflict, ledger the ruling" [label="yes"];
    "Rule on the conflict, ledger the ruling" -> "Fix round R of 5: R≤3 resume implementer; R≥4 fresh implementer, more capable model";
    "Finding conflicts with plan text?" -> "Fix round R of 5: R≤3 resume implementer; R≥4 fresh implementer, more capable model" [label="no"];
    "Fix round R of 5: R≤3 resume implementer; R≥4 fresh implementer, more capable model" -> "Dispatch scoped re-review (./re-review-prompt.md)";
    "Dispatch scoped re-review (./re-review-prompt.md)" -> "All findings addressed?";
    "All findings addressed?" -> "Update plan checkboxes in task commit,\nresolve final HEAD,\nappend completion to ledger,\nmark todo complete" [label="yes"];
    "All findings addressed?" -> "R = 5?" [label="no"];
    "R = 5?" -> "Fix round R of 5: R≤3 resume implementer; R≥4 fresh implementer, more capable model" [label="no - next round"];
    "R = 5?" -> "Adjudicate each open finding" [label="yes - breaker trips"];
    "Adjudicate each open finding" -> "Any load-bearing finding?";
    "Any load-bearing finding?" -> "Rule and continue; stop only if every path forward is a guess" [label="yes"];
    "Any load-bearing finding?" -> "Park findings in ledger with rulings" [label="no"];
    "Park findings in ledger with rulings" -> "Update plan checkboxes in task commit,\nresolve final HEAD,\nappend completion to ledger,\nmark todo complete";
    "Update plan checkboxes in task commit,\nresolve final HEAD,\nappend completion to ledger,\nmark todo complete" -> "More tasks remain?";
    "More tasks remain?" -> "Dispatch implementer subagent (./implementer-prompt.md)" [label="yes"];
    "More tasks remain?" -> "Dispatch final code reviewer (../requesting-code-review/code-reviewer.md)" [label="no"];
    "Dispatch final code reviewer (../requesting-code-review/code-reviewer.md)" -> "Final findings? ONE fix dispatch, one scoped re-review, adjudicate residuals";
    "Final findings? ONE fix dispatch, one scoped re-review, adjudicate residuals" -> "Final review clean: harvest ledger findings to the project backlog, commit, then delete this plan's workspace";
    "Final review clean: harvest ledger findings to the project backlog, commit, then delete this plan's workspace" -> "Use superpowers:finishing-a-development-branch";
}
```

## Setup

Ensure the work happens in an isolated workspace: use
superpowers:using-git-worktrees to create one or verify the existing one.
Never start implementation on a main/master branch without your human
partner's explicit consent.

Conversation memory does not survive compaction. In real sessions,
controllers that lost their place have re-dispatched entire completed task
sequences — the single most expensive failure observed. Track progress in
the plan-scoped ledger and the committed plan file, not only in todos.

- Each plan owns a workspace: at skill start, run this skill's
  `scripts/sdd-workspace PLAN_FILE` — it prints the plan's git-ignored
  directory (`<repo-root>/.superpowers/sdd/<plan-basename>/`), home to
  every artifact for THIS plan: ledger, briefs, reports, review packages.
  Another plan's directory is never yours to read or write.
- Check for this plan's ledger at `<workspace>/progress.md`. If its first
  line names your plan file, tasks with a `Task <N>: complete` line are DONE
  — do not re-dispatch them; resume at the first task without one. A task
  whose last line is a fix round is mid-loop: resume the loop at the next
  round. A ledger whose first line names a different plan file — or a stray
  ledger at the old flat path `.superpowers/sdd/progress.md` — is another
  plan's progress: leave it in place and start your own, fresh.
- Create the ledger with its identity as the first line:
  `# SDD ledger — plan: <plan file path>`.
- Also read the plan file's checkboxes. A checked-off task is durable resume
  state for later sessions. If the ledger and plan disagree, stop and
  reconcile them with `git log` before dispatching more work.
- The ledger is your recovery map: the commits it names exist in git even
  when your context no longer remembers creating them. The committed plan
  is the cross-session progress map. After compaction, trust the ledger,
  plan checkboxes, and `git log` over your own recollection.
- `git clean -fdx` will destroy the workspace (it's git-ignored scratch); if
  that happens, recover from the committed plan checkboxes and `git log`.

Read the plan once, note its context and Global Constraints, and create a
todo per task. If the plan names a Spec, read that too: the spec is the
authority the plan argues from, and conflicts inside the plan resolve
against it. A plan with no reachable spec gets a ledger note saying so —
rulings made without one are provisional.

Before dispatching Task 1, scan the plan once for conflicts, writing down
what you checked as you check it:

- tasks that contradict each other or the plan's Global Constraints
- anything the plan explicitly mandates that the review rubric treats as a
  defect (a test that asserts nothing, verbatim duplication of a logic block)
- **plan text the design has since superseded.** If the plan cites a design or
  spec doc, check whether that doc carries decisions added or revised _after_
  the plan was drafted — look for dated revisions, "added after review",
  "supersedes", or a decision numbered past the ones the plan quotes. Plan
  prose describing a superseded decision is the highest-risk text in the
  document: it reads as authoritative, implementers transcribe it verbatim, and
  every scoped reviewer approves it for matching the brief. Note which tasks
  quote it. In a real session, a decision reversed after design approval left
  its old form in the plan's own docs task; the wrong claim reached three
  source files and three docs before a whole-branch sweep caught it.
- **the plan's literals and citations, checked against the codebase.** The
  three checks above read the plan against itself and its docs; this one reads
  it against reality. Run the plan's fixture values through the real library
  (a FEN through the chess engine, a date through the parser — an invalid
  fixture often degrades a test silently rather than erroring). Grep for every
  helper, import path, and signature the plan cites; confirm each exists under
  that name with that arity. Where a step's prose and its sample code
  disagree, the prose is usually the intent and the code is the bug. In a real
  session a plan was wrong six times — invalid FEN fixtures that would have
  made a test assert nothing, an import that didn't exist, a cache callback
  that evicted the wrong entry — and four of the six were mechanically
  detectable in about five minutes of this check.
- **snippets that claim to mirror an existing pattern.** Where a task says
  "same contract as X" or "like the existing Y", diff the task's sample code
  against the real pattern it names. The literal check cannot catch this:
  every symbol can exist while the snippet drops a guard every sibling
  carries. In a real session a count snippet claiming "same contract as the
  existing chip counts" omitted the soft-delete guard that every sibling
  count wraps — it would have counted deleted rows in a user-facing chip,
  and was caught only because the controller happened to hand the
  implementer "the actual contract wins" as a constraint.
- **literals only the outside world can confirm.** Account names, live
  endpoints, external IDs — the codebase cannot vouch for any of them, so
  the repo-literal check above never sees them. Give each one cheap probe
  (a curl) at pre-flight, or carry an explicit "unverified external
  literal" flag into the dispatch of the task that consumes it. In a real
  session a plan paired a username with the wrong provider; the error
  survived all ten task reviews into the live smoke — where the adapter's
  correct 404 briefly read as a failure — and got transcribed into a unit
  test's provider pairing.
- **external-world assumptions the plan defers to a later task.** When a
  later step says to verify something against a live service but hedges it
  ("informational; do not gate the commit on it"), the assumption will
  ship verified by nothing. If the service is reachable, verify it NOW
  with a few read-only calls and record what came back. In a real session
  four pre-flight curls confirmed — among other deferred assumptions — the
  missing-record response shape an entire branch of the diff logic
  depended on; had it been wrong, live user data would have been
  duplicated, and the plan's own verification step gated nothing.
- **shared types widened in one task, consumed in another.** For each task
  that widens a shared type or enum, ask: does the type-check gate pass at
  THIS task's commit with consumers untouched? Widening first breaks the
  widening task's own gate — the widened inferred type no longer assigns
  to the narrower unions consumers still declare — so the widening belongs
  in the same task as its first consumer, or later. The literal check
  verifies the symbols exist; it never simulates the compile fallout of a
  cross-task type split. In a real session a plan put an enum widening in
  Task 1 and its three consumers in Task 8, asserting the schema "already
  carries it"; Task 1's own check gate broke and the change had to be
  re-homed by controller ruling.

The scan's output is a table, not a verdict. One row for every pair of tasks
that share a file or an interface: the two tasks, what one produces against
what the other consumes, and what you found. One row for every task: whether
its own text agrees with itself — the tests it specifies against the code it
specifies, the files it creates against the files it later touches. "The scan
is clean" without those rows is not a scan you ran.

Write the table to the ledger. Rule on everything you find before execution
begins — each finding against the plan text that mandates it — and record
each ruling in the ledger. If the scan is clean, proceed without comment.
Rule on each conflict it surfaces — the spec is the binding authority, the
plan is its argument — record the ruling beside its row, and dispatch
Task 1. The review loop remains the net for conflicts that only emerge from
implementation.

## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

**Mechanical implementation tasks** (isolated functions, clear specs, 1-2 files): use a fast, cheap model. Most implementation tasks are mechanical when the plan is well-specified.

**Integration and judgment tasks** (multi-file coordination, pattern matching, debugging): use a standard model.

**Architecture and design tasks**: use the most capable available model.
The final whole-branch review is one of these — dispatch it on the most
capable available model, not the session default.

**Review tasks**: choose the model with the same judgment, scaled to the
diff's size, complexity, and risk. A small mechanical diff does not need the
most capable model; a subtle concurrency change does. Scoped re-reviews of
small fix diffs take a cheap-to-mid tier.

**Fix-loop escalation (rounds 4-5)**: use a model at least one tier above
the implementer that got stuck.

**Always specify the model explicitly when dispatching a subagent.** An
omitted model inherits your session's model — often the most capable and
most expensive — which silently defeats this section.

**Turn count beats token price.** Wall-clock and context cost scale with how
many turns a subagent takes, and the cheapest models routinely take 2-3× the
turns on multi-step work — costing more overall. Use a mid-tier model as the
floor for reviewers and for implementers working from prose descriptions.
When the task's plan text contains the complete code to write, the
implementation is transcription plus testing: use the cheapest tier for
that implementer. Single-file mechanical fixes also take the cheapest tier.

**Task complexity signals (implementation tasks):**

- Touches 1-2 files with a complete spec → cheap model
- Touches multiple files with integration concerns → standard model
- Requires design judgment or broad codebase understanding → most capable model

## The Task Loop

**Batch small same-shape work.** When the plan lists several tasks that are
each a small, independent edit of the same kind — the same one-line fix,
constant change, or field addition repeated across files — do not dispatch
one subagent per task. Compose ONE dispatch brief listing every file and
its change, send the whole batch to a single subagent, and review its diff
as one unit. Reserve one-dispatch-per-task for work that needs its own
judgment, its own tests, or its own review surface.

Everything you paste into a dispatch prompt — and everything a subagent
prints back — stays resident in your context for the rest of the session
and is re-read on every later turn. Hand artifacts over as files.

**Waiting on dispatched subagents:** never poll a wait interface with
short timeouts, and never sit in one silent, open-ended wait either.
While you have local work — ledger updates, packaging the next review,
reading reports — keep working; child results arrive on their own.
When you are genuinely idle, wait in bounded stretches (five to ten
minutes, where your platform allows), and between stretches post one
line of status and reconcile your live children: list them, and chase
any that finished without reporting. A bounded stretch keeps nearly
all of a long wait's efficiency while guaranteeing a stuck or lost
child is noticed within minutes, not at the end of the session.

### 1. Dispatch the implementer

Record BASE (`git rev-parse HEAD`) before dispatching — the review package
and fix-round diffs need it. Before every dispatch, BASE must equal the
previous task's recorded final head from the ledger (for Task 1, the head
setup left). Unexpected drift means another writer — a concurrent session,
another tool — is on this checkout: STOP and investigate authorship before
dispatching anything. Attribute mystery commits by their content (e.g. a
plan bug present verbatim in a commit whose dispatch had corrected it),
not by commit trailers — legitimate commits lack trailers too. Never
dispatch into a moved tree.

- **Task brief:** before dispatching an implementer, run this skill's
  `scripts/task-brief PLAN_FILE N` — it extracts the task's full text to a
  uniquely named file and prints the path. Compose the dispatch so the
  brief stays the single source of
  requirements. Your dispatch should contain: (1) one line on where this
  task fits in the project; (2) the brief path, introduced as "read this
  first — it is your requirements, with the exact values to use verbatim";
  (3) interfaces and decisions from earlier tasks that the brief cannot
  know; (4) your resolution of any ambiguity you noticed in the brief;
  (5) the report-file path and report contract. Exact values (numbers,
  magic strings, signatures, test cases) appear only in the brief. Never
  make a subagent read the whole plan file.
- **Report file:** name the implementer's report file after the brief
  (brief `…/task-N-brief.md` → report `…/task-N-report.md`) and put it in
  the dispatch prompt. The implementer writes the full report there and
  returns only status, commits, a one-line test summary, and concerns.
- A dispatch prompt describes one task, not the session's history. Do not
  paste accumulated prior-task summaries ("state after Tasks 1-3") into
  later dispatches — a real session's dispatch hit 42k chars of which 99%
  was pasted history. A fresh subagent needs its task, the interfaces it
  touches, and the global constraints. Nothing else.
- The dispatch carries the no-subagents contract (it is in the
  implementer template): the implementer never dispatches subagents —
  not helpers, and never a reviewer. Review arrives from you, after the
  report. In real sessions, every reviewer a worker spawned duplicated
  the task review the controller dispatched anyway — a full extra
  review seat per task.
- If an earlier task parked a finding in the area this task touches, carry
  a pointer to that ledger entry in the dispatch.
- Record the implementer's agent identity from the dispatch result —
  fix-loop rounds 1-3 resume this agent.
- Never dispatch multiple implementation subagents in parallel (conflicts).

Template: [implementer-prompt.md](implementer-prompt.md)

### 2. Handle the report

Implementer subagents report one of four statuses. Handle each appropriately:

**DONE:** Generate the review package (`scripts/review-package PLAN_FILE BASE HEAD`, from this skill's directory — it prints the unique file path it wrote; BASE is the commit you recorded before dispatching the implementer — never `HEAD~1`, which silently drops all but the last commit of a multi-commit task), then dispatch the task reviewer with the printed path.

**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts. Read the concerns before proceeding. If the concerns are about correctness or scope, address them before review. If they're observations (e.g., "this file is getting large"), note them and proceed to review.

**NEEDS_CONTEXT:** The implementer needs information that wasn't provided. Provide the missing context and re-dispatch.

**BLOCKED:** The implementer cannot complete the task. Assess the blocker:

1. If it's a context problem, provide more context and re-dispatch with the same model
2. If the task requires more reasoning, re-dispatch with a more capable model
3. If the task is too large, break it into smaller pieces
4. If the plan itself is wrong, rule on the correction, ledger it, and re-dispatch with the ruling carried in the dispatch

**Never** ignore an escalation or force the same model to retry without changes. If the implementer said it's stuck, something needs to change.

**A "pre-existing failure" claim requires proof.** When any report attributes
a test failure to the baseline — "environmental", "flaky", "was already
failing" — do not accept the claim on the implementer's word. Run the same
test file in a clean worktree at the pre-task SHA (`git worktree add <tmp>
<sha>`) and quote both results side by side; only a failure reproduced there
is pre-existing. In a real session an implementer reported full-suite
failures as environmental pollution; the clean worktree showed 15 pass /
0 fail before their diff and 8 failures on it — a real single-file
regression, minutes from being accepted as noise.

If the implementer asks questions — before starting or mid-task — answer
clearly and completely, provide additional context if needed, and don't
rush it into implementation.

### 3. Review the task

Per-task reviews are task-scoped gates. The broad review happens once, at the
final whole-branch review. Never skip the task review, and never accept a
report missing either verdict — spec compliance AND task quality are both
required. Implementer self-review never replaces the task review; both are
needed.

- Hand the reviewer its diff as a file: run this skill's
  `scripts/review-package PLAN_FILE BASE HEAD` and pass the reviewer the file path
  it prints (or, without bash: `git log --oneline`, `git diff --stat`,
  and `git diff -U10` for the range, redirected to one uniquely named
  file). The output never enters your own context, and the reviewer sees
  the commit list, stat summary, and full diff with context in one Read
  call. Use the BASE you recorded before dispatching the implementer —
  never `HEAD~1`, which silently truncates multi-commit tasks. Never
  dispatch a task reviewer without a diff file.
- **Reviewer inputs:** the task reviewer gets three paths — the same brief
  file, the report file, and the review package — plus the global
  constraints that bind the task.
- The global-constraints block you hand the reviewer is its attention
  lens. Copy the binding requirements verbatim from the plan's Global
  Constraints section or the spec: exact values, exact formats, and the
  stated relationships between components ("same layout as X", "matches
  Y"). The reviewer's template already carries the process rules (YAGNI,
  test hygiene, review method) — the constraints block is for what THIS
  project's spec demands.
- Do not add open-ended directives like "check all uses" or "run race tests
  if useful" without a concrete, task-specific reason
- Do not ask a reviewer to re-run tests the implementer already ran on the
  same code — the implementer's report carries the test evidence
- Do not pre-judge findings for the reviewer — never instruct a reviewer to
  ignore or not flag a specific issue. If you believe a finding would be a
  false positive, let the reviewer raise it and adjudicate it in the review
  loop. If the prompt you are writing contains "do not flag," "don't treat X
  as a defect," "at most Minor," or "the plan chose" — stop: you are
  pre-judging, usually to spare yourself a review loop.
  The task reviewer may report "⚠️ Cannot verify from diff" items — requirements
  that live in unchanged code or span tasks. These do not block the rest of the
  review, but you must resolve each one yourself before marking the task
  complete: you hold the plan and cross-task context the reviewer
  lacks. If you confirm an item is a real gap, treat it as a failed spec
  review — it enters the fix loop with the other findings.

Template: [task-reviewer-prompt.md](task-reviewer-prompt.md)

### 4. The fix loop

The loop triggers when the review reports spec ❌, any Critical or Important
finding, or a ⚠️ item you confirmed as a real gap.

Before the loop starts, two routes leave it immediately:

- Record Minor findings in the progress ledger as you go
  (`Task <N>: minor (deferred): <one-liner>`), and point the final
  whole-branch review at that list so it can triage which must be fixed
  before merge. A roll-up nobody reads is a silent discard. Minor findings
  never enter the loop. Write each one to be readable months later by someone
  without this session: name the file, and say what is wrong rather than that
  something is. Triage decides which get fixed; Finish's harvest step is what
  keeps the rest — the ledger itself is git-ignored and does not survive.
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
Model Selection), with the brief path, the report-file path, the open
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
required is present. Name the
covering test files in the fix message — a one-line fix does not need the
whole suite.

**The re-review is scoped.** Run `scripts/review-package PLAN_FILE FIX_BASE HEAD`
where FIX_BASE is the head the previous review saw, and dispatch
[re-review-prompt.md](re-review-prompt.md) with the findings list, the
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

### 5. Complete the task

When the review comes back clean — or every open finding is parked with a
ruling at the cap — first flip that task's plan checkboxes
(`- [ ]` -> `- [x]`) and fold the plan file update into the task's
implementation commit, not a separate bookkeeping commit. Prefer staging the
plan file with the task's files; if the task was already committed, confirm
the worktree is otherwise clean, stage only the plan file, and run
`git commit --amend --no-edit`.

After that task commit is final, resolve its new `HEAD`, then append the
completion line to the ledger. `<head7>` must be the first seven characters
of this final, post-amend `HEAD`:

- `Task <N>: complete (commits <base7>..<head7>, review clean)`
- `Task <N>: complete (commits <base7>..<head7>, <K> parked)` after a
  tripped breaker

Mark the todo complete only after the ledger, plan file, and task commit agree,
then move on. Never move to the next task while the review has open
Critical/Important issues that are neither fixed nor parked-with-ruling at the
cap.

## Final Review

The final whole-branch review gets a package too: run
`scripts/review-package PLAN_FILE MERGE_BASE HEAD` (MERGE_BASE = the commit the
branch started from, e.g. `git merge-base main HEAD`) and include the
printed path in the final review dispatch, so the final reviewer reads
one file instead of re-deriving the branch diff with git commands. Dispatch
on the most capable available model (see Model Selection), using
superpowers:requesting-code-review's
[code-reviewer.md](../requesting-code-review/code-reviewer.md). Point it at
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

If the final whole-branch review returns findings, dispatch ONE fix subagent
with the complete findings list — not one fixer per finding.
Per-finding fixers each rebuild context and re-run suites; a real
session's final-review fix wave cost more than all its tasks combined.
Then run exactly one scoped re-review of the fix wave
(`scripts/review-package PLAN_FILE FIX_BASE HEAD` over the fix range,
[re-review-prompt.md](re-review-prompt.md)).
Adjudicate any residual findings as in the task loop's breaker: park with
rulings, or rule on the load-bearing ones and ledger what you decided. Only
the four classes above stop you here. There is no second fix wave —
residual load-bearing findings surface to your human partner when
finishing-a-development-branch presents the options.

## Finish

### 1. Harvest the ledger — before deleting anything

Before you delete anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, breaker adjudications, all of them — into
your final message under "Rulings I made", in the order you made them, each
with what it costs if wrong. The list is exhaustive: if the ledger holds a
ruling, the list holds it. That list is the only place the decisions you
took on your human partner's behalf reach them — they read it and rework
whatever you got wrong. A ruling that dies with the workspace was a decision
made in secret.

Git holds the code and the fixes. It does **not** hold the findings you chose
not to fix: deferred minors, parked findings with their rulings, accepted
deviations, plan defects you ruled on, and anything marked UNREVIEWED. Those
exist only in a git-ignored file you are about to delete. Deleting the
workspace without harvesting is the silent discard this skill forbids
everywhere else.

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

### 2. Delete the workspace

Only once the harvest is committed: `rm -rf <workspace>`. Sibling directories
belong to other plans; leave them alone.

### 3. Hand off

Use superpowers:finishing-a-development-branch. Carry any UNREVIEWED ledger
entry into that conversation as a named caveat — it is your human partner's
call whether to merge work no second pair of eyes has seen.

## Common Rationalizations

| Excuse                                                 | Reality                                                                                                                                                                          |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Close enough on spec compliance"                      | Reviewer found spec gaps = not done. Fix or hit the cap and adjudicate — those are the only exits.                                                                               |
| "I'll fix it myself, dispatching is overhead"          | Controller fixes pollute your context and skip review. Resume the implementer.                                                                                                   |
| "One more round will converge"                         | Past the cap, rounds don't converge — the failure is structural. Adjudicate and route.                                                                                           |
| "The reviewer will just find something new anyway"     | Scoped re-reviews verify fixes; they cannot wander. New findings on untouched code go to the ledger, not the loop.                                                               |
| "This finding is obviously wrong, I'll drop it"        | You adjudicate only at the cap, and every ruling is a ledger entry. Silent discards are forbidden.                                                                               |
| "The fix was small, skip the re-review"                | Unreviewed fixes are how regressions land. Every round ends with a scoped re-review.                                                                                             |
| "Reviews slow the loop down"                           | The loop without reviews is just unverified churn. Reviews are the loop's brakes and steering.                                                                                   |
| "Durable-state bookkeeping is overhead"                | The plan-scoped ledger survives compaction; the committed plan survives across sessions. Controllers without both have re-dispatched completed tasks or lost visible progress.   |
| "The implementer spawned its own reviewer — free extra assurance" | It's a duplicate seat reviewing the same diff; the task review is the gate. A worker-spawned reviewer is a defect to flag, not rigor.                                  |
| "The git history is the record — delete the workspace" | Git has the code and the fixes. It never had the findings you chose _not_ to fix. Harvest first, then delete.                                                                    |
| "The final review already triaged the minors"          | Triage picks which to fix. The ones it declines still need a committed home, or the `rm -rf` discards them.                                                                      |
| "Every task review passed, so the branch is clean"     | Scoped reviewers cannot see a defect whose symptom lives in a file no task touched. That is what the cross-cutting sweeps are for.                                               |
| "The plan says it, so it's correct"                    | Plans are written before the code and go stale when a decision changes after drafting. Doc prose in a plan is the likeliest thing to be wrong and the least likely to be caught. |
| "That test failure was already there"                  | A pre-existing-failure claim without a clean-worktree run at the pre-task SHA is an alibi, not a finding. Reproduce it there or treat it as a regression.                        |
| "Dispatch is down, so I'll just fix it and move on"    | You may fix it. You may not call it reviewed. Commit body, ledger `UNREVIEWED` line, discriminating-test proof, and a named caveat at finish — all four.                         |

## Example Workflow

```
You: I'm using Subagent-Driven Development to execute this plan.

[Setup: worktree verified]
[Read plan file once: docs/superpowers/plans/feature-plan.md]
[Resolve workspace: scripts/sdd-workspace docs/superpowers/plans/feature-plan.md — no ledger inside, fresh start]
[Create todos for all tasks]

Task 1: Hook installation script

[Run task-brief for Task 1; dispatch implementer with brief + report paths + context]

Implementer: "Before I begin - should the hook be installed at user or system level?"

You: "User level (~/.config/superpowers/hooks/)"

Implementer: [Later]
  - Implemented install-hook command
  - Added tests, 5/5 passing
  - Self-review: Found I missed --force flag, added it
  - Committed

[Run review-package PLAN_FILE BASE HEAD; dispatch task reviewer with the printed path]
Task reviewer: Spec ✅ - all requirements met, nothing extra.
  Strengths: Good test coverage, clean. Issues: None. Task quality: Approved.

[Flip Task 1 plan checkboxes and stage them with Task 1's commit, or amend that commit before Task 2]
[Resolve the final post-amend HEAD]
[Ledger: Task 1: complete (commits a1b2c3d..<final-head7>, review clean)]

Task 2: Recovery modes

[Run task-brief for Task 2; dispatch implementer with brief + report paths + context]

Implementer: [No questions]
  - Added verify/repair modes
  - 8/8 tests passing
  - Committed

[Run review-package PLAN_FILE BASE HEAD; dispatch task reviewer with the printed path]
Task reviewer: Spec ❌:
  - Missing: Progress reporting (spec says "report every 100 items")
  Issues (Important): Magic number (100)

[Fix round 1: resume the implementer with both findings]
Implementer: Added progress reporting, extracted PROGRESS_INTERVAL constant.
  Re-ran test/recovery.test.js — 10/10 passing. Fix report appended.

[Run review-package PLAN_FILE FIX_BASE HEAD; dispatch scoped re-review]
Re-reviewer: Missing progress reporting — ADDRESSED (src/recovery.js:41).
  Magic number — ADDRESSED (src/recovery.js:7). New breakage: none.
  Verdict: all findings addressed.

[Ledger: Task 2: fix round 1/5 (2 addressed, 0 open; commits d4e5f6a..b7c8d9e)]
[Flip Task 2 plan checkboxes and stage them with Task 2's commit, or amend that commit before Task 3]
[Resolve the final post-amend HEAD]
[Ledger: Task 2: complete (commits d4e5f6a..<final-head7>, review clean)]

...

[After all tasks]
[Run review-package PLAN_FILE MERGE_BASE HEAD; dispatch final code-reviewer, most capable model]
Final reviewer: All requirements met. Deferred minors triaged: none block merge.

[Harvest the ledger's deferred/parked findings into the project's backlog, commit]
[Delete this plan's workspace — the code is in git, the findings are in the backlog]

Done! Using superpowers:finishing-a-development-branch.
```
