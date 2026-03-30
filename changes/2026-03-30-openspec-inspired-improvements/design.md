# OpenSpec-Inspired Improvements

Three modifications to existing superspec skills, inspired by comparing with OpenSpec's workflow flexibility.

## 1. Schema Flexibility (ss-brainstorming)

### Problem

Every change currently requires the full flow: design.md + plan.md + tasks.md. For well-understood changes (bug fixes, adding a similar endpoint, small refactors), the design doc step adds friction without proportional value.

### Design

Add a complexity tier assessment after "Explore project context" and before clarifying questions. The agent proposes **full** or **light** based on scope signals, and the user confirms via AskUserQuestion.

**Complexity signals for light:**
- Single file or small scope
- Bug fix with clear reproduction
- Well-understood pattern (e.g., "add endpoint like existing ones")
- User describes the solution, not just the problem

**Complexity signals for full:**
- Multiple components or subsystems
- Unclear trade-offs or multiple viable approaches
- New patterns not in the codebase
- User describes the problem, needs exploration

**If light (user confirms):**
- Skip checklist items 4-7 (propose approaches, present design, write design doc, spec review loop)
- Go from clarifying questions directly to invoking ss-writing-plans
- ss-writing-plans produces plan.md + tasks.md without a design.md reference

**If full:**
- Current flow, unchanged

**Impact on ss-writing-plans:** The plan header's design.md reference becomes optional. If no design.md exists, the plan is based on conversation context and the user's description.

### Supported flows after this change

```
Flow 1: Full (default)
  ss-brainstorming -> [infers "full"] -> design.md
    -> ss-writing-plans -> plan.md + tasks.md
      -> execution -> ss-archive

Flow 2: Light (well-understood changes)
  ss-brainstorming -> [infers "light", user confirms]
    -> ss-writing-plans -> plan.md + tasks.md (no design.md)
      -> execution -> ss-archive
```

## 2. Fluid Workflow (ss-subagent-driven-development & ss-executing-plans)

### Problem

The current flow is hard-gated: design -> plan -> execute. When implementation reveals a design problem, the agent has no protocol for handling it. It either gets stuck or silently deviates.

### Design

Add a "Design Deviation Protocol" to both execution skills. Triggers when implementation reveals a mismatch with the plan or design.

**Triggers:**
- Implementer reports a blocker from incorrect plan assumptions
- Executor discovers planned approach won't work (API doesn't exist, dependency conflict, architectural constraint)
- A task's spec doesn't match what the codebase needs

**NOT a design deviation (handle inline):**
- Minor implementation details (variable names, exact line numbers)
- Test adjustments for framework quirks

**The protocol:**

1. **Pause** the current task
2. **Surface** to user: what was expected (from plan/design) vs what was found, and why it matters
3. **User decides** via AskUserQuestion:
   - **Update artifacts and continue** — Agent edits design.md and/or plan.md, adds note `> Updated during implementation: [reason]`, reviews and adjusts remaining tasks, then resumes
   - **Proceed as-is** — Continue with pragmatic fix, agent notes the deviation in tasks.md as a comment
   - **Rethink** — Drop back to brainstorming-level discussion

**Skill-specific integration:**

- **ss-subagent-driven-development:** Design deviations surface through the existing `BLOCKED` and `DONE_WITH_CONCERNS` implementer statuses. The controller applies this protocol when the blocker is a design mismatch (not a context problem).
- **ss-executing-plans:** The executor detects deviations during task execution and applies the same protocol.

## 3. Archive Expansion — Project Documentation Sync (ss-archive)

### Problem

Archive produces a knowledge artifact, but the knowledge stays isolated in `changes/archive/`. Project-level documentation (README, API docs, architecture docs) doesn't get updated, gradually drifting from reality.

### Design

Add a "Project Documentation Sync" step after generating the knowledge artifact.

**New step (between current Step 4 "Save to Archive" and Step 5 "Report"):**

1. **Analyze the change** — Read change artifacts to understand what was built, what capabilities were added/changed
2. **Scan for affected project docs** — Look for documentation files that might need updating:
   - Files referenced in the change artifacts
   - Well-known doc files (README.md, CHANGELOG, docs/) whose content overlaps with the change
   - Files describing features/APIs/architecture that the change touched
3. **Present findings** — Show user which files were detected and what update each needs
4. **User confirms** — AskUserQuestion with multiSelect: which files to update (including "None, skip")
5. **Edit confirmed files** — Surgical updates reflecting the change, not doc rewrites

**Constraints:**
- Only updates existing files (never creates new doc files)
- Surgical edits only (not section rewrites)
- Always requires user confirmation
- If no affected docs found, skip silently

**Updated archive flow:**

```
Step 1: Locate change
Step 2: Read change artifacts
Step 3: Generate knowledge artifact
Step 4: Project Documentation Sync (NEW)
Step 5: Report
```
