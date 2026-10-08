---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write implementation plans for an engineer who has not seen this codebase or this spec. Assume they write idiomatic code in the project's language once they know the exact interface and the exact test, and that they will make a reasonable choice wherever the plan leaves one open. What they cannot know is what you decided: which files, which names and signatures, which values from the spec, which tests prove each task. Document those. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

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

If the spec leaves product decisions open, interview your human partner before writing tasks — plans built on guessed product calls get rewritten. When a question hinges on a product object that doesn't exist in the app yet (something this feature would create), ground it per the brainstorming skill's visual-companion guide, section "Ground in the Real Product": real screenshots of the current app, mockups only for the delta, labeled real-vs-mock, and the proposed logic dry-run against the user's real data. A plan pre-flight that must verify literals anyway often doubles as this interview material. The same grounding applies when your partner's reply shows confusion about how something already behaves: reach for the brief on the FIRST confused reply — a second prose explanation of the same mechanism is the wrong move.

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

## Step Granularity

**Each step is one action with a checkable result:**
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

## Review Focus

[The five input classes or failure modes the spec implies but no task's
tests exercise that are most likely to bite a person using this software
— one line each, naming the input or condition and the behavior a
reasonable person would expect, most likely first. The spec is a vision
document: it says what the software must do, not everything it will
meet, and its silence on an input is not permission for that input to
break the program. Write the list here, once, with the spec in front of
you. Then, for each line, add the test that pins it to the task that
owns the code, in that task's own step style.]

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
  (Subagent-driven) or superpowers:executing-plans (Native) — the one you
  recommend for this plan in the Execution Handoff, or the method your
  human partner already named.
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

- [ ] **Step 3: Implement `function(input: InputType) -> ResultType` in `exact/path/to/file.py`**

One line on the approach when the signature and the test leave a choice
(which library call, which data structure); a code block only for an
algorithm they do not determine.

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## What a Step Contains

A step is done when the implementer can write exactly one reasonable thing
from it. That is the whole requirement: unambiguous, not complete. Each kind
of step carries what makes it unambiguous and nothing more:

- **A test step:** the test's name and its assertions, as code, with the
  spec's exact values in them.
- **A code step:** the exact signature (name, parameters, return type), the
  file it lives in, and the specific values the spec pins. The implementer
  writes the body. A body appears only for an algorithm the signature and
  tests do not determine, or for exact copy the spec fixes.
- **A verification step:** the command to run and the output that means it
  passed.
- **A reference to another task:** that task's Interfaces block says what
  to use; the plan does not repeat that task's code.

A plan is the set of decisions the implementer cannot make alone. A plan
longer than the code it describes has written the code instead. Lines that
decide nothing ("TBD", "handle edge cases", "add appropriate validation",
"write tests for the above", a type or function no task defines) are the
opposite failure, and the self-review catches both.

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Step scan:** Every step must let the implementer write exactly one reasonable thing, and no step may carry more than that: a line that decides nothing is a gap, a function body the signature and tests already determine is a transcript. Fix both.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. Review Focus:** For each input class or failure mode the spec implies, is there a task whose tests exercise it? The five uncovered ones most likely to bite a person go in the Review Focus section, and each line there gets its test added to the owning task. An empty section means you checked and found none, not that you skipped the check.

**5. Proportion:** Compare the plan's length to the spec's. A plan several times longer than the spec it implements is a transcript of the program, not a plan. If code blocks are most of the document, replace bodies with signatures, test names and assertions, and check that each step is still unambiguous.

**6. Reality check against the codebase:** The checks above read the plan against itself and the spec; this one reads it against the repo. Run every fixture literal through the real library it targets (an invalid fixture often degrades a test silently instead of erroring). Grep for every existing helper, import path, and signature the plan cites — confirm each exists under that name with that arity. Implementers transcribe the plan's literals, signatures and test assertions verbatim and reviewers approve them for matching the brief, so a wrong literal here survives every later gate.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

The plan already ends with a `## Session Handoff` section written during
plan creation. Never spend a turn generating handoff content here — no
subagent dispatch, no re-reading files, no invoking another skill. By the
time you reach this point, the handoff exists.

After saving and self-reviewing the plan, end with an informational
message — NOT a blocking question. Your human partner reviews the plan and
chooses how it runs, either by replying here or by launching the starter
prompt in a fresh session; nothing executes in this session before that
reply. Recommend the executor that fits this plan. **Default to Native.**
Recommend Subagent-driven only when the plan widens a type, enum or
interface that later tasks or other code consume, or runs past ~8 tasks.
On a measured 3-task plan, Native cost about a third as much, took half
the time, and its one final review caught a plan bug the per-task
reviewers passed.

- **Subagent-driven** - A fresh subagent implements each task and a fresh reviewer checks it before the next one starts, then a whole-branch review at the end. Most thorough; costs a fresh context per task and per review.
- **Native** - The executing session implements every task itself, the way this harness runs work, then one fresh reviewer on the most capable model checks the whole branch. Cheapest and fastest; no independent review until the end. Runs well with a mid-tier session model, since the plan carries the design.

**When no execution method has already been supplied,** the message is, in
order:

**"Plan saved to `<plan path>` — please review it before execution starts.
For this plan I recommend <Subagent-driven | Native>, because <one sentence
from the plan: how much the tasks depend on each other's interfaces, how
many there are, what a shipped mistake would cost>."**

Then the starter prompt from the plan's Session Handoff section, in a
fenced block, copyable straight from the terminal — it names the executor
you recommend. Then:

**"Start a fresh session with the prompt above, or say 'go'
(subagent-driven) / 'native' (executing-plans) to run it here."**

**When an execution method has already been supplied,** keep it: no
recommendation line, and the starter prompt names that method's executor.
The message is the plan path and "please review it before execution
starts", the starter prompt, then **"Start a fresh session with the prompt
above, or say '<go | native>' to run it here."** — the word for the method
they named.

If your human partner starts a fresh session, this session is done. A reply
of 'go' or 'native' here is their plan review and execution-method choice;
launching the starter prompt carries the same choice into the fresh
session.

**If Subagent-driven chosen ('go'):**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development

**If Native chosen ('native'):**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
