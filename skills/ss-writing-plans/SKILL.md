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
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step

**Note:** Do NOT include git commit steps in plans. The user handles their own git workflow.

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
- No git write commands (commit, push, merge) — the user handles git

## Self-Review

After writing the complete plan, look at it with fresh eyes against the spec (if any). This is a checklist you run yourself inline — not a subagent dispatch. Fix issues directly; no re-review needed.

**1. Spec coverage.** If `design.md` exists, skim each requirement and point at the task that implements it. List any gaps and add tasks to close them. If no spec, check the plan against the user's stated goals.

**2. Placeholder scan.** Search the plan for red flags. These are **plan failures** — never ship them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases" without showing how
- "Write tests for the above" without actual test code
- "Similar to Task N" — repeat the code; the engineer may read tasks out of order
- References to types, functions, or methods not defined in any task

**3. Type / signature consistency.** Do the names, types, and signatures used in later tasks match what was defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug. Fix it.

**4. Buildability.** Read each task as if you have zero project context. Could you follow it without getting stuck? If a step describes *what* without showing *how*, expand it.

**Calibration:** flag only issues that would cause an implementer to build the wrong thing or get stuck. Minor wording is not an issue.

## Tasks.md Generation

After writing the plan, generate a `tasks.md` file in the same change directory. This file extracts the high-level task checklist from the plan:

```markdown
# Tasks: [Feature Name]

**Change:** changes/YYYY-MM-DD-<topic>/
**Plan:** plan.md
**Design:** design.md *(omit this line if no design.md exists)*

## Tasks

- [ ] Task 1: [Component Name] — [one-line summary]
- [ ] Task 2: [Component Name] — [one-line summary]
- [ ] Task 3: [Component Name] — [one-line summary]
...
```

Each line corresponds to a `### Task N` section in the plan. Execution skills will mark these `- [x]` as tasks complete, providing persistent cross-session tracking.

## Execution Handoff

After saving the plan and tasks.md:

- Announce: "Plan complete and saved to `changes/YYYY-MM-DD-<topic>/plan.md`. Tasks tracked in `tasks.md`."
- Then decide the execution mode based on the **One-shot signal**:

**If invoked with `mode: one-shot`** (handoff from ss-brainstorming One-shot, or user explicitly states one-shot):
- Skip the execution-mode prompt — default to **Subagent-Driven**
- This preserves the One-shot promise of a continuous pass (design → plan → tasks → execution without intermediate prompts)
- User still gets the bundled review gate before execution actually starts (back in ss-brainstorming)

**Otherwise (Standard mode or no signal)** — use AskUserQuestion with these options:
- **Subagent-Driven (Recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration
- **Inline Mode** — Execute tasks directly in this session, batch execution with checkpoints

**REQUIRED SUB-SKILL:** Use ss-subagent-driven-development (subagent mode or inline mode)
