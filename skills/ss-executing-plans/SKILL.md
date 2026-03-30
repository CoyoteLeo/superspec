---
name: ss-executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the ss-executing-plans skill to implement this plan."

<GIT-GUARDRAIL>
Do NOT execute git write commands (commit, push, merge, rebase, branch delete) directly. The user handles their own git workflow. You may run read-only git commands (status, log, diff) to gather information.
</GIT-GUARDRAIL>

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use ss-subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

**Dual tracking:** Use both TodoWrite (ephemeral, in-session) and `tasks.md` (persistent, in change directory). When marking a task complete, update both. If resuming from a prior session, read `tasks.md` first to see what's already done.

For each task:
1. Mark as in_progress (TodoWrite)
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed (TodoWrite AND `tasks.md`: `- [ ]` → `- [x]`)

### Step 3: Complete Development

After all tasks complete and verified:
1. Run the project test suite to confirm everything passes
2. Suggest: "All tasks complete. Consider running `/superspec:ss-archive` to capture the architectural knowledge from this change."

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly
- Plan assumptions don't match reality (API doesn't exist, dependency conflict) — use **Design Deviation Protocol**

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Design Deviation Protocol

When you discover during task execution that the plan's assumptions don't match reality, apply this protocol.

**What counts as a design deviation:**
- Plan assumes an API/dependency/pattern that doesn't exist or works differently
- Task's approach conflicts with the actual codebase architecture
- A requirement from the design is impossible or impractical to implement as specified

**What does NOT count (handle inline):**
- Minor implementation details (variable names, exact line numbers)
- Test adjustments for framework quirks
- Small scope adjustments that don't change the design intent

**The protocol:**

1. **Pause** the current task
2. **Surface** to the user: what was expected (from plan/design) vs what was found, and why it matters
3. **User decides** via AskUserQuestion:
   - **Update artifacts and continue** — Edit design.md and/or plan.md, add note `> Updated during implementation: [reason]`, review and adjust remaining tasks, then resume
   - **Proceed as-is** — Continue with the pragmatic fix, note the deviation in tasks.md as a comment below the task checkbox
   - **Rethink** — Drop back to brainstorming-level discussion about the approach

After the user decides and any artifact updates are made, resume execution from where you paused.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **ss-writing-plans** - Creates the plan this skill executes

**Suggested after completion:**
- **ss-archive** - Capture architectural knowledge from the change
