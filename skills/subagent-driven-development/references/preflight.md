# The Pre-flight Scan

Part of superpowers:subagent-driven-development. Run it once, before
dispatching Task 1.

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

| Excuse | Reality |
|--------|---------|
| "The plan says it, so it's correct" | Plans are written before the code and go stale when a decision changes after drafting. Doc prose in a plan is the likeliest thing to be wrong and the least likely to be caught. |
