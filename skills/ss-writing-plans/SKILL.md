---
name: ss-writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the ss-writing-plans skill to create the implementation plan."

**Context:** Invoked by ss-brainstorming after the design phase. Both Standard and One-shot modes produce a `design.md` before handing off — use it as the primary reference. The change directory should already exist. In One-shot mode, plan + tasks generation happens in the same continuous pass as the design (no separate user gate between them); the user reviews all three artifacts together afterwards.

**Save plans to:** `changes/YYYY-MM-DD-<topic>/plan.md` (in the change directory)
- (User preferences for plan location override this default)

**Also generate:** `changes/YYYY-MM-DD-<topic>/tasks.md` — a standalone task checklist extracted from the plan for cross-session persistence. Each task is a `- [ ]` checkbox line with a brief description. This file is the persistent tracking mechanism that survives across conversations (unlike TodoWrite which is ephemeral).

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- **A file's boundary is "who uses it", not "how big it is."** Things only one consumer uses belong in that consumer's file. Splitting a single-consumer unit into five files to keep each one short doesn't reduce coupling — it hides the coupling in the import graph, where it is harder to see and easier to break.
- **Split something out when a second consumer genuinely shares it**, and then make it a real single source of truth rather than a pass-through layer.
- Files that change together should live together. Split by responsibility, not by technical layer.
- A file that has become hard to read is a prompt to ask "is it actually serving two consumers?" — if yes, split along that seam; if no, leave it whole.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying is serving two unrelated consumers, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

**Also state the PR grouping.** Say which tasks land in which PR, what each PR's base is (a stacked PR is based on the previous PR's branch, not on the base branch), and any ordering constraint with its reason — e.g. a backend PR must merge before the frontend PR that calls its new endpoint. The execution skill needs this to open the right PRs; leaving it implicit means the stack gets guessed.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step

**Note:** Do NOT put git steps inside a task's steps. Branching, committing, pushing and opening the PR happen at the PR-group boundary and are the controller's job (see ss-subagent-driven-development → *Worktrees and Stacked PRs*), not a step an implementer subagent performs. Task steps stay code + test only.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use ss-subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

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

````

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD
- No git steps inside tasks — the PR flow lives at the group boundary, and the user is the one who merges

## Subagent Review

After writing the complete plan, dispatch a `general-purpose` subagent to review it against the spec (if any). Inline self-review is **not** acceptable — empirically the author rubber-stamps their own work and skips codebase verification. The reviewer must open the repo to verify concrete claims (file paths, symbol names, fixtures, signatures, that referenced types/functions actually exist).

Use the template at `plan-document-reviewer-prompt.md`. Pass the absolute path of `plan.md` and, if it exists, `design.md`.

When the subagent returns:
- **Approved with no issues** → proceed to tasks.md generation.
- **Issues found** → fix each one (or, for an issue where you disagree, note your reasoning in `plan.md` so the user can adjudicate), then re-dispatch the reviewer on the updated file. Loop until approved.

**Calibration:** the reviewer is instructed to flag only issues that would cause an implementer to build the wrong thing or get stuck. Minor wording is not an issue.

## Tasks.md Generation

After writing the plan, generate a `tasks.md` file in the same change directory. This file extracts the high-level task checklist from the plan:

```markdown
# Tasks: [Feature Name]

**Change:** changes/YYYY-MM-DD-<topic>/
**Plan:** /abs/path/to/plan.md
**Design:** design.md *(omit this line if no design.md exists)*

## Execution State

*(scaffold this block empty — the execution skill fills it in)*

**Worktree:** _not started_
**PRs:** _none yet_

## Tasks

- [ ] Task 1: [Component Name] — [one-line summary]
- [ ] Task 2: [Component Name] — [one-line summary]
- [ ] Task 3: [Component Name] — [one-line summary]
...
```

Each task line corresponds to a `### Task N` section in the plan. The execution skill marks these `- [x]`, annotates each one with its commit range and review verdict, and keeps the Execution State block current — together that makes `tasks.md` the change's record of what has actually happened, not just what is left.

**The `Plan:` line is absolute.** After execution starts, `tasks.md` is read from a worktree, from another terminal, or in a session that has forgotten where it came from — a repo-relative path resolves to the wrong file or to nothing.

**Do not split this into a second progress file.** One file per change is the record; a sibling ledger duplicates the task list and the two drift.

## Execution Handoff

After saving the plan and tasks.md:

- Announce: "Plan complete and saved to `/abs/path/to/changes/YYYY-MM-DD-<topic>/plan.md`. Tasks tracked in `tasks.md`." Use the **absolute** path — the user often opens these from a different terminal or worktree.
- Then hand off. **There is no execution-mode question** — execution is always subagent-driven, in a worktree, shipping as stacked PRs.

**REQUIRED SUB-SKILL:** Use ss-subagent-driven-development.
