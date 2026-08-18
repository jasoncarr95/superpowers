---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`

- (Repo conventions override this default: if the repo's CLAUDE.md/AGENTS.md
  names a planning-docs location — e.g.
  `notes/project-planning/<feature>/YYYY-MM-DD-<slug>-execution-plan.md` —
  save there instead. Explicit user preferences override both.)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## Open Product Calls

If the spec leaves product decisions open, interview your human partner before writing tasks — plans built on guessed product calls get rewritten. When a question hinges on a product object that doesn't exist in the app yet (something this feature would create), ground it per the brainstorming skill's visual-companion guide, section "Ground in the Real Product": real screenshots of the current app, mockups only for the delta, labeled real-vs-mock, and the proposed logic dry-run against the user's real data. A plan pre-flight that must verify literals anyway often doubles as this interview material.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

## Documentation Tasks Specify Facts, Not Sentences

A plan is written before the code exists, so its prose goes stale the moment a
decision changes after drafting — and it goes stale _invisibly_. An implementer
transcribes a paragraph faithfully; a task reviewer approves it because it
matches the brief. Nothing in the loop compares that paragraph to the code that
actually shipped.

So a documentation task lists **the facts the docs must end up asserting** and
**which file must assert each one** — never the paragraphs to paste. If the
project has a docs-updating skill or command, invoking it is the task's first
step, and the fact list becomes the verification checklist rather than the input.

- Bad: a fenced block of finished Markdown for the implementer to copy.
- Good: "AGENTS.md must state that resolution reads the navigated line and
  builds no graph, and that the counts are the graph-backed exception. Derive
  the wording from the shipped code, not from this plan."

In a real session, a plan shipped a paragraph asserting a design rule that a
late revision had already reversed. It reached three source files and three
docs and passed every scoped review, because every reviewer was checking
transcription accuracy against the plan — the one document that was wrong.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**

- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking; update the checkboxes as work completes and commit those updates with the task work so a later session can resume from the plan without reconstructing progress from chat.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the spec/design doc this plan implements — the plan
argues from the spec, so the spec travels with it; executors read both]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## Plan Document Footer: Session Handoff

**Every plan MUST end with a `## Session Handoff` section**, written in the
same turn as the rest of the plan and committed with it — never as a separate
step or a follow-up turn. It exists so your human partner can close this
session and execute the plan in a fresh one without losing anything. Two
parts, in order:

**1. Starter prompt** — a fenced block your human partner copies to launch
the fresh session. Derive the wording from this plan (facts, not fixed
sentences). It must convey:

- The plan document's path.
- The spec/design document's path (omit if none exists).
- The executor skill to invoke: superpowers:subagent-driven-development
  (or superpowers:executing-plans if this plan calls for inline execution).
- An instruction to read the plan's Session Handoff section first, then
  begin at the first unchecked task, updating checkboxes as work completes.

**2. "Don't forget" list** — up to ~8 bullets restricted to items that do
NOT belong in the plan itself:

- Manual steps your human partner must do personally (accounts, approvals,
  hardware).
- Ideas explicitly deferred out of scope during planning.
- Environment gotchas discovered during planning that don't affect any task.
- Post-implementation follow-ups (docs elsewhere, people to tell, projects
  this unblocks).

Never pad — one real item beats three invented ones. If nothing qualifies,
write the single line: "None — everything is captured in the plan and spec."

**If an item would change how any task is implemented, it goes into the
plan, not the handoff.** The handoff is a pointer plus reminders — never a
second copy of plan content.

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**

- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**

- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:

- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. Reality check against the codebase:** The three checks above read the plan against itself; this one reads it against the repo. Run every fixture literal through the real library it targets (an invalid fixture often degrades a test silently instead of erroring). Grep for every existing helper, import path, and signature the plan cites — confirm each exists under that name with that arity. Implementers transcribe plan code verbatim and reviewers approve it for matching the brief, so a wrong literal here survives every later gate.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

The plan already ends with a `## Session Handoff` section written during
plan creation. Never spend a turn generating handoff content here — no
subagent dispatch, no re-reading files, no invoking another skill. By the
time you reach this point, the handoff exists.

After saving the plan, end with an informational message — NOT a blocking
question — containing:

1. The starter prompt from the plan's Session Handoff section, in a fenced
   block, copyable straight from the terminal.
2. The execution options, requiring no reply for the fresh-session path:

**"Plan saved to `<plan path>`. Recommended: start a fresh session with the
prompt above — it executes the plan with subagent-driven development on a
clean context. Or, to execute here instead, say 'go' (subagent-driven) or
'inline' (executing-plans)."**

If your human partner starts a fresh session, this session is done. If they
reply here instead:

**If 'go' (Subagent-Driven):**

- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If 'inline' (Inline Execution):**

- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
