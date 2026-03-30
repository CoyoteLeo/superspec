# OpenSpec-Inspired Improvements Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use ss-subagent-driven-development (recommended) or ss-executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add schema flexibility, fluid workflow deviation handling, and project doc sync to superspec's skill pipeline.

**Architecture:** Modify 5 existing skill markdown files plus 1 prompt template. No new skills created. Changes are additive sections inserted into existing skill structures.

**Tech Stack:** Markdown skill files (no runtime code)

---

### Task 1: Schema Flexibility — ss-brainstorming

**Files:**
- Modify: `skills/ss-brainstorming/SKILL.md`

Add a complexity tier assessment step between "Explore project context" (item 1) and "Offer visual companion" (item 2). The agent infers full/light and the user confirms.

- [ ] **Step 1: Add complexity tier section**

Insert a new section `## Complexity Tier` after the `## Anti-Pattern` section and before `## Checklist`. Content:

```markdown
## Complexity Tier

After exploring project context, assess whether the change needs a **full** or **light** flow, and propose the tier to the user.

**Signals for light:**
- Single file or small scope change
- Bug fix with clear reproduction
- Well-understood pattern (e.g., "add endpoint like existing ones")
- User describes the solution, not just the problem

**Signals for full (default):**
- Multiple components or subsystems involved
- Unclear trade-offs or multiple viable approaches
- New patterns not yet in the codebase
- User describes the problem, needs design exploration

**How it works:** After exploring project context, propose the tier using AskUserQuestion:
> "This looks like a [light/full] change because [reason]. Which tier?"
> - **Full** — Design doc with approach exploration, then plan (default)
> - **Light** — Skip design doc, go straight to planning

**If light (user confirms):**
- Skip checklist items 5-9 (propose approaches, present design, write design doc, spec review loop, user reviews spec)
- Go from clarifying questions directly to invoking ss-writing-plans
- No change directory is created during brainstorming — ss-writing-plans creates it

**If full:**
- Current flow, unchanged
```

- [ ] **Step 2: Update checklist to reference tier**

In the `## Checklist` section, insert a new item 2 after "Explore project context" and before "Offer visual companion":

```markdown
2. **Assess complexity tier** — infer full/light, propose to user via AskUserQuestion (see Complexity Tier section)
```

Renumber all subsequent items (current 2 becomes 3, current 3 becomes 4, etc. — total becomes 10 items).

- [ ] **Step 3: Add light-flow skip note to checklist**

After the renumbered checklist, add:

```markdown
**Light flow:** If user confirmed light tier in step 2, skip items 5-9 and go directly from item 4 (clarifying questions) to item 10 (transition to implementation).
```

- [ ] **Step 4: Update process flow diagram**

Update the `digraph brainstorming` to include a "Complexity tier?" diamond after "Explore project context" that branches to either "Visual questions ahead?" (full) or directly to "Ask clarifying questions" → "Invoke ss-writing-plans" (light).

- [ ] **Step 5: Verify the modified file**

Read the full modified `skills/ss-brainstorming/SKILL.md` and verify:
- Checklist items are correctly numbered 1-10
- Light flow skip note references correct item numbers
- Process flow diagram is valid dot syntax
- No contradictions with existing content

- [ ] **Step 6: Commit**

```bash
git add skills/ss-brainstorming/SKILL.md
git commit -m "feat: add complexity tier (full/light) to ss-brainstorming"
```

---

### Task 2: Schema Flexibility — ss-writing-plans

**Files:**
- Modify: `skills/ss-writing-plans/SKILL.md`
- Modify: `skills/ss-writing-plans/plan-document-reviewer-prompt.md`

Make design.md reference conditional so ss-writing-plans works in both full and light flows.

- [ ] **Step 1: Update Context section**

Replace the current `**Context:**` paragraph (line 19):

```markdown
**Context:** This should be run in a dedicated worktree (created by ss-brainstorming skill). The ss-brainstorming skill creates a change directory at `changes/YYYY-MM-DD-<topic>/` with a `design.md` inside it.
```

With:

```markdown
**Context:** Invoked by ss-brainstorming after the design phase (full flow) or directly after clarifying questions (light flow). If a `design.md` exists in the change directory, use it as the primary reference. Otherwise, base the plan on conversation context and the user's description. If the change directory doesn't exist yet (light flow), create it: `changes/YYYY-MM-DD-<topic>/`.
```

- [ ] **Step 2: Update tasks.md template**

In the `## Tasks.md Generation` section, update the template to make the Design line conditional:

```markdown
# Tasks: [Feature Name]

**Change:** changes/YYYY-MM-DD-<topic>/
**Plan:** plan.md
**Design:** design.md *(omit this line if no design.md exists)*

## Tasks

- [ ] Task 1: [Component Name] — [one-line summary]
...
```

- [ ] **Step 3: Update plan-document-reviewer-prompt.md**

In `skills/ss-writing-plans/plan-document-reviewer-prompt.md`, update the prompt template to make the spec reference optional:

Replace:
```
    **Spec for reference:** [SPEC_FILE_PATH]
```

With:
```
    **Spec for reference:** [SPEC_FILE_PATH or "No design doc — plan based on conversation context"]
```

- [ ] **Step 4: Verify both modified files**

Read the full modified files and verify:
- Context section clearly describes both full and light flows
- tasks.md template conditional is clear
- Plan reviewer prompt handles missing spec gracefully
- No contradictions with existing content

- [ ] **Step 5: Commit**

```bash
git add skills/ss-writing-plans/SKILL.md skills/ss-writing-plans/plan-document-reviewer-prompt.md
git commit -m "feat: make design.md optional in ss-writing-plans for light flow"
```

---

### Task 3: Design Deviation Protocol — ss-subagent-driven-development

**Files:**
- Modify: `skills/ss-subagent-driven-development/SKILL.md`

Add the Design Deviation Protocol, integrated with existing implementer status handling.

- [ ] **Step 1: Add Design Deviation Protocol section**

Insert a new section `## Design Deviation Protocol` after the `## Handling Implementer Status` section and before `## Prompt Templates`. Content:

```markdown
## Design Deviation Protocol

When an implementer's `BLOCKED` or `DONE_WITH_CONCERNS` status indicates a **design mismatch** (not a context problem), apply this protocol instead of the normal status handling.

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

After the user decides and any artifact updates are made, resume the normal per-task flow from where it was paused.
```

- [ ] **Step 2: Add cross-reference in Handling Implementer Status**

In the existing `**BLOCKED:**` section, add a note after point 4:

```markdown
5. If the blocker is a design mismatch (plan assumptions don't match reality), apply the **Design Deviation Protocol** below instead
```

In the existing `**DONE_WITH_CONCERNS:**` section, update the text to reference the protocol:

```markdown
**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts. Read the concerns before proceeding. If the concerns indicate a design mismatch (plan assumptions vs reality), apply the **Design Deviation Protocol**. If the concerns are about correctness or scope, address them before review. If they're observations (e.g., "this file is getting large"), note them and proceed to review.
```

- [ ] **Step 3: Add to Red Flags**

In the `## Red Flags` section, under `**Never:**`, add:

```markdown
- Silently deviate from the plan without surfacing the mismatch (use Design Deviation Protocol)
- Decide to update or skip artifacts on behalf of the user (always escalate)
```

- [ ] **Step 4: Verify the modified file**

Read the full modified `skills/ss-subagent-driven-development/SKILL.md` and verify:
- Design Deviation Protocol section is placed correctly
- Cross-references in Handling Implementer Status are accurate
- Red flags additions don't duplicate existing entries
- No contradictions with existing content

- [ ] **Step 5: Commit**

```bash
git add skills/ss-subagent-driven-development/SKILL.md
git commit -m "feat: add design deviation protocol to ss-subagent-driven-development"
```

---

### Task 4: Design Deviation Protocol — ss-executing-plans

**Files:**
- Modify: `skills/ss-executing-plans/SKILL.md`

Add the same Design Deviation Protocol, adapted for inline execution.

- [ ] **Step 1: Add Design Deviation Protocol section**

Insert a new section `## Design Deviation Protocol` after `## When to Revisit Earlier Steps` and before `## Remember`. Content:

```markdown
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
```

- [ ] **Step 2: Update "When to Stop and Ask for Help"**

In the existing `**STOP executing immediately when:**` list, add:

```markdown
- Plan assumptions don't match reality (API doesn't exist, dependency conflict) — use **Design Deviation Protocol**
```

- [ ] **Step 3: Verify the modified file**

Read the full modified `skills/ss-executing-plans/SKILL.md` and verify:
- Design Deviation Protocol section is placed correctly
- Cross-reference in "When to Stop" is accurate
- No contradictions with existing content

- [ ] **Step 4: Commit**

```bash
git add skills/ss-executing-plans/SKILL.md
git commit -m "feat: add design deviation protocol to ss-executing-plans"
```

---

### Task 5: Project Documentation Sync — ss-archive

**Files:**
- Modify: `skills/ss-archive/SKILL.md`

Add the Project Documentation Sync step between "Save to Archive" and "Report".

- [ ] **Step 1: Add Project Documentation Sync step**

Insert a new `### Step 5: Project Documentation Sync` after the existing `### Step 4: Save to Archive` section. Content:

```markdown
### Step 5: Project Documentation Sync

Update project-level documentation to reflect what this change built.

1. **Analyze the change** — Read the change artifacts (design.md, plan.md, tasks.md) to understand what capabilities were added or changed
2. **Scan for affected project docs** — Look for existing documentation files that might need updating:
   - Files referenced in the change artifacts (e.g., "modifies the auth flow documented in docs/auth.md")
   - Well-known doc files (README.md, CHANGELOG, docs/) whose content overlaps with the change
   - Files describing features, APIs, or architecture that the change touched
3. **Present findings** — Show user which files were detected and what kind of update each needs (e.g., "README.md — add mention of new light flow option")
4. **User confirms** — Use AskUserQuestion with multiSelect: which files to update. Include a "None, skip this step" option.
5. **Edit confirmed files** — Make surgical updates reflecting the change. Keep edits minimal and focused — update existing content, don't rewrite sections.

**Constraints:**
- Only update existing files (never create new doc files)
- Surgical edits only (not section rewrites)
- If no affected docs found, skip silently — don't ask "no docs found, want to create some?"
```

- [ ] **Step 2: Renumber existing Step 5 to Step 6**

Rename the existing `### Step 5: Report` to `### Step 6: Report`.

- [ ] **Step 3: Update the Report step**

In the renamed Step 6 Report, update the output template to include doc sync info:

```markdown
**Docs updated:** [list of files updated, or "None"]
```

Add this line after the existing `**Source:**` line in the report template.

- [ ] **Step 4: Verify the modified file**

Read the full modified `skills/ss-archive/SKILL.md` and verify:
- Steps are numbered 1-6 consecutively
- Project Documentation Sync step is between Save to Archive and Report
- Report template includes doc sync info
- Guardrails section still accurate
- No contradictions with existing content

- [ ] **Step 5: Commit**

```bash
git add skills/ss-archive/SKILL.md
git commit -m "feat: add project documentation sync to ss-archive"
```
