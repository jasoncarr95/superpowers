# Handling a Report That Isn't DONE

Part of superpowers:subagent-driven-development.

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

| Excuse | Reality |
|--------|---------|
| "That test failure was already there" | A pre-existing-failure claim without a clean-worktree run at the pre-task SHA is an alibi, not a finding. Reproduce it there or treat it as a regression. |
