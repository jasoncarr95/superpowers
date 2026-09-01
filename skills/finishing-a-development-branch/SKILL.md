---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work
---

# Finishing a Development Branch

## Overview

**Core principle:** Verify tests → Update docs → Detect environment → Resolve the integration choice → Execute it → Clean up → Wrap the session, all in one turn.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## Step 1: Verify Tests

Run the project's full test suite (`npm test` / `cargo test` / `pytest` / `go test ./...`).

**If tests fail**, report the failures and stop — the menu comes after a green suite:

```
Tests failing (<N> failures). Must fix before completing:

[Show failures]
```

**If tests pass:** continue to Step 2.

## Step 2: Update the Docs (before integration)

Docs work happens **in the branch, before merge** — never as a post-merge
chore. Before presenting the integration menu:

- If the project declares a docs-before-merge convention (check AGENTS.md /
  CLAUDE.md for it — e.g. an `/update-docs` pass covering changelog
  `[Unreleased]` entries, a decision-board refresh, deferred-idea harvest,
  and planning-doc archiving at a chunk's final PR), run it now.
- Otherwise cover the minimum by hand: a changelog entry if the project
  keeps one, todo/backlog updates for follow-ups discussed but not
  implemented, and a session handoff if open work remains.
- Commit the docs updates on this branch before continuing.

If the docs pass already ran this session (e.g. SDD's harvest step just
completed and the docs are current), verify `git status` is clean and move
on — don't duplicate it.

## Step 3: Detect Environment

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# Capture now, while still inside the workspace — Step 6 changes directory
# before cleanup (Step 7) needs this value
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

This determines which menu to show and how cleanup works:

| State                                  | Integration choice                          | Cleanup                             |
| -------------------------------------- | ------------------------------------------- | ----------------------------------- |
| `GIT_DIR == GIT_COMMON` (normal repo)  | PR default; standard 3-option menu fallback | No worktree to clean up             |
| `GIT_DIR != GIT_COMMON`, named branch  | PR default; standard 3-option menu fallback | Provenance-based (see Step 6)       |
| `GIT_DIR != GIT_COMMON`, detached HEAD | Reduced 2-option menu (no merge)            | Externally managed — leave in place |

## Step 4: Determine Base Branch

The base branch is whatever this work forked from — usually named in the
plan, the conversation, or the branch's upstream. If it is not already
known, ask: "This branch split from <your best guess> - is that correct?"
Confirm before merging: merging into the wrong base is expensive to undo.

## Step 5: Resolve the Integration Choice

Take the first of these that applies:

1. **Your human partner already named the choice** in this conversation
   ("push and open a PR", "merge it locally", "leave the branch for now").
   Execute that. Their words win over every default below.
2. **Fork preference (standing, no need to re-ask):** on a named branch in
   a repo with a remote (`git remote get-url origin` succeeds), the
   integration choice is **Option 2 — push and create a Pull Request**.
   Execute it without presenting the menu. The decision was made once, as
   a standing preference; re-asking it at every finish only inserts a turn
   boundary between the integration step and the wrap.
3. **Otherwise present the menu:** detached HEAD, no remote, or a partner
   instruction you cannot map to an option. Ask it through the
   AskUserQuestion tool when this environment has one, so the answer comes
   back inside this turn and the remaining steps run without a hand-off;
   without that tool, print the menu and wait.

**Normal repo and named-branch worktree — present exactly these 3 options:**

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)

Which option?
```

**Detached HEAD — present exactly these 2 options:**

```
Implementation complete. You're on a detached HEAD (externally managed workspace).

1. Push as new branch and create a Pull Request
2. Keep as-is (I'll handle it later)

Which option?
```

When a menu is due, present it exactly as written — concise, with every
option coming from the list above — and wait for the answer. Merging
locally and discarding are never defaults: a local merge happens only when
your human partner names it, and discarding only in response to their
explicit request (see "If your human partner asks to discard the work"
below).

## Step 6: Execute Choice

### Option 1: Merge Locally

```bash
# Get main repo root for CWD safety
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# Merge first — verify success before removing anything
git checkout <base-branch>
git pull
git merge <feature-branch>

# Verify tests on merged result
<test command>
```

If tests fail on the merged result: stop, leave the worktree and branch in
place, and investigate — nothing has been pushed, so the merge is local
and recoverable.

Once the merged result is green: clean up the worktree (Step 7), then
delete the branch:

```bash
git branch -d <feature-branch>
```

Then continue to Step 8 in this same turn.

### Option 2: Push and Create PR

```bash
git push -u origin <feature-branch>
# From a detached HEAD, name the new branch on the remote:
# git push origin HEAD:refs/heads/<new-branch>
```

Then create the pull/merge request against <base-branch> with the forge's
tooling — its CLI if one is available, or the creation URL most forges
print when you push — following the repo's PR template and conventions if
present, and report the URL to your human partner.

Keep the worktree — your human partner iterates on PR feedback there.
Then continue to Step 8 in this same turn — the PR URL is not the end of
the finish.

### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>." Then
continue to Step 8 in this same turn.

### If your human partner asks to discard the work

This path exists only as a response to an explicit request to throw the
work away. Confirm first:

```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

Wait for that exact confirmation. When it arrives:

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

Then clean up the worktree (Step 7) and force-delete the branch:

```bash
git branch -D <feature-branch>
```

## Step 7: Cleanup Workspace

**Runs for Option 1 and confirmed discards.** Options 2 and 3 always
preserve the worktree. Both callers have already changed directory to the
main repo root — worktree removal must run from outside the worktree —
and use the `GIT_DIR`/`GIT_COMMON`/`WORKTREE_PATH` values captured in
Step 3, from before that directory change.

**If `GIT_DIR == GIT_COMMON`:** Normal repo, no worktree to clean up. Done.

**If `WORKTREE_PATH` is under `.worktrees/` or `worktrees/`:** Superpowers
created this worktree — we own cleanup:

```bash
git worktree remove "$WORKTREE_PATH"
git worktree prune  # Self-healing: clean up any stale registrations
```

**If removal is refused** (`contains modified or untracked files`): the
worktree holds files that exist nowhere else — uncommitted plans, notes,
or scratch work. Never `--force` on your own initiative. Show your human
partner what is at stake and ask:

```bash
git -C "$WORKTREE_PATH" status --porcelain -uall
```

```
Worktree removal refused — these files were never committed:

<file list>

1. Commit them to <branch> before cleanup
2. Move them into <main repo root>
3. Delete them (unrecoverable)

Which?
```

Carry out the choice, then remove the worktree.

**Otherwise:** The host environment owns this workspace — leave it in
place. If your platform provides a workspace-exit tool, use it.

## Step 8: Session Wrap

After the integration choice has executed (any option), and in the same
turn — the wrap is part of finishing, not a follow-up your human partner
has to ask for — close the loop on session knowledge:

- If a `wrap-session` skill is available in this environment, invoke it —
  it audits durability, emits a fresh-session starter prompt, and runs a
  process retro.
- Otherwise, when open work remains (PR awaiting merge, deferred
  follow-ups, a planned next chunk), end your final message with a fenced
  copy-paste starter prompt for the next session — pointers, not prose:
  the goal in one sentence, preconditions to verify first, exact files to
  read, and any constraint a fresh session cannot infer.

## Quick Reference

| Option                          | Merge | Push | Keep Worktree | Cleanup Branch |
| ------------------------------- | ----- | ---- | ------------- | -------------- |
| 1. Merge locally (on request)   | yes   | -    | -             | yes            |
| 2. Create PR (standing default) | -     | yes  | yes           | -              |
| 3. Keep as-is (on request)      | -     | -    | yes           | -              |
| Discard (explicit request only) | -     | -    | -             | yes (force)    |

## Common Rationalizations

| Excuse                                                        | Reality                                                                                                                                    |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| "Tests passed earlier this session"                           | Run the suite on the tree you are about to integrate. A green run only proves the tree it ran on.                                          |
| "Docs can wait until after the merge"                         | Docs-before-merge is the convention: changelog, board refresh, and harvest run in the branch (Step 2). Post-merge docs chores get skipped. |
| "They obviously want it merged"                               | Merging and discarding happen only in your human partner's words. The standing default is push + PR; nothing else runs unasked.            |
| "Better to ask than assume — I'll show the menu anyway"       | The PR default is a standing decision recorded here. Re-asking inserts the turn boundary that drops the wrap. Menu only for the fallbacks.  |
| "The PR is up — the wrap can wait until they ask"             | Step 8 runs in the same turn as the integration step. A finish that stops at the PR URL is unfinished.                                     |
| "They seem done with this feature — I'll offer to discard it" | The menu is complete as written. Discard happens only when your human partner asks for it in so many words.                                |
| "'Yeah, get rid of it' counts as confirmation"                | Only the typed word `discard` authorizes deletion.                                                                                         |
| "The PR is up, so the worktree is clutter now"                | PR feedback gets fixed in that worktree. It stays until the work lands.                                                                    |
| "This other worktree looks stale — I'll clean it too"         | Clean up only worktrees under `.worktrees/` or `worktrees/`. Everything else belongs to the host.                                          |
| "Removal refused — `--force` is just finishing the cleanup"   | The refusal means files exist only in that worktree. `--force` destroys them permanently. Show your human partner and ask.                 |
| "The merged-result failure is probably flaky"                 | A failing merged result stops everything. Branch and worktree stay put while you investigate.                                              |
| "The base branch is obviously main"                           | Confirm the fork point or ask. Merging into the wrong base is expensive to undo.                                                           |
| "The push was rejected — force-push will fix it"              | A rejected push means the remote moved. Investigate; force-push only on your human partner's explicit request.                             |
