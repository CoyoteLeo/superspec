# Simplify Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use ss-subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reduce superspec from 13 skills to 5 by removing general-purpose discipline skills and merging redundant execution/review skills.

**Architecture:** Delete 6 skill directories, merge 2 skills into ss-subagent-driven-development, update all cross-references in remaining skills and README.

**Tech Stack:** Markdown skill files (no runtime code)

---

## File Structure

**After this change, the skills/ directory will contain:**

```
skills/
  ss-archive/SKILL.md
  ss-brainstorming/SKILL.md, spec-document-reviewer-prompt.md, visual-companion.md, scripts/
  ss-subagent-driven-development/SKILL.md, implementer-prompt.md, spec-reviewer-prompt.md, code-quality-reviewer-prompt.md, code-reviewer.md (moved from ss-requesting-code-review)
  ss-using-superspec/SKILL.md, references/
  ss-writing-plans/SKILL.md, plan-document-reviewer-prompt.md
agents/
  code-reviewer.md
  code-simplifier.md
```

**Deleted directories (8):**
- `ss-test-driven-development/` (general practice)
- `ss-systematic-debugging/` (general practice)
- `ss-verification-before-completion/` (general practice)
- `ss-receiving-code-review/` (team preferences)
- `ss-writing-skills/` (meta-skill for authors)
- `ss-dispatching-parallel-agents/` (built into Claude Code)
- `ss-executing-plans/` (merged into ss-subagent-driven-development)
- `ss-requesting-code-review/` (merged into ss-subagent-driven-development)

---

### Task 1: Merge ss-executing-plans into ss-subagent-driven-development

**Files:**
- Modify: `skills/ss-subagent-driven-development/SKILL.md`
- Read: `skills/ss-executing-plans/SKILL.md` (source for merge)

Absorb ss-executing-plans as an "Inline Mode" section in ss-subagent-driven-development.

- [ ] **Step 1: Read ss-executing-plans/SKILL.md**

Read the full content to understand what needs to be preserved. Key elements:
- Load and review plan critically
- Dual tracking (TodoWrite + tasks.md)
- Execute tasks sequentially
- When to stop and ask for help
- Design Deviation Protocol (already added in previous change)

- [ ] **Step 2: Add Inline Mode section to ss-subagent-driven-development**

Insert a new section `## Inline Mode` after the `## Integration` section (end of file). Content:

```markdown
## Inline Mode

When subagents are unavailable or impractical, execute the plan directly in the current session.

**Use inline mode when:**
- Platform doesn't support subagents
- Tasks are tightly coupled and need shared context
- User explicitly requests inline execution

**The process:**

1. Read plan file, review critically — raise concerns before starting
2. Create TodoWrite with all tasks
3. For each task:
   - Mark as in_progress (TodoWrite)
   - Follow each step exactly
   - Run verifications as specified
   - Mark as completed (TodoWrite AND tasks.md: `- [ ]` → `- [x]`)
4. After all tasks: run full test suite, suggest ss-archive

**When to stop:** Hit a blocker, plan has gaps, instruction unclear, verification fails repeatedly. Ask for clarification rather than guessing.

**Design Deviation Protocol applies** — same as subagent mode (see above).
```

- [ ] **Step 3: Update the "When to Use" diagram**

In the existing `digraph when_to_use`, replace the `ss-executing-plans` node with `Inline mode (see below)` and update the edge label from "no - parallel session" to "no subagents or tightly coupled".

- [ ] **Step 4: Update Integration section**

Remove the line:
```
- **ss-executing-plans** - Use for parallel session instead of same-session execution
```

- [ ] **Step 5: Verify and commit**

Read the full modified file. Verify no broken references remain.

```bash
git add skills/ss-subagent-driven-development/SKILL.md
git commit -m "feat: absorb ss-executing-plans as inline mode in ss-subagent-driven-development"
```

---

### Task 2: Merge ss-requesting-code-review into ss-subagent-driven-development

**Files:**
- Move: `skills/ss-requesting-code-review/code-reviewer.md` → `skills/ss-subagent-driven-development/code-reviewer.md`
- Modify: `skills/ss-subagent-driven-development/code-quality-reviewer-prompt.md`

The code-reviewer.md template is referenced by code-quality-reviewer-prompt.md. Move the file and update the reference.

- [ ] **Step 1: Move code-reviewer.md**

```bash
cp skills/ss-requesting-code-review/code-reviewer.md skills/ss-subagent-driven-development/code-reviewer.md
```

- [ ] **Step 2: Update reference in code-quality-reviewer-prompt.md**

In `skills/ss-subagent-driven-development/code-quality-reviewer-prompt.md`, replace:
```
  Use template at ss-requesting-code-review/code-reviewer.md
```
With:
```
  Use template at ./code-reviewer.md
```

- [ ] **Step 3: Update Integration section in SKILL.md**

In `skills/ss-subagent-driven-development/SKILL.md`, remove the line:
```
- **ss-requesting-code-review** - Code review template for reviewer subagents
```

- [ ] **Step 4: Verify and commit**

```bash
git add skills/ss-subagent-driven-development/code-reviewer.md skills/ss-subagent-driven-development/code-quality-reviewer-prompt.md skills/ss-subagent-driven-development/SKILL.md
git commit -m "feat: absorb ss-requesting-code-review template into ss-subagent-driven-development"
```

---

### Task 3: Delete removed skill directories

**Files:**
- Delete: `skills/ss-test-driven-development/` (2 files)
- Delete: `skills/ss-systematic-debugging/` (12 files)
- Delete: `skills/ss-verification-before-completion/` (1 file)
- Delete: `skills/ss-receiving-code-review/` (1 file)
- Delete: `skills/ss-writing-skills/` (7 files)
- Delete: `skills/ss-dispatching-parallel-agents/` (1 file)
- Delete: `skills/ss-executing-plans/` (1 file, already merged)
- Delete: `skills/ss-requesting-code-review/` (2 files, template already moved)

- [ ] **Step 1: Delete all 8 directories**

```bash
git rm -r skills/ss-test-driven-development
git rm -r skills/ss-systematic-debugging
git rm -r skills/ss-verification-before-completion
git rm -r skills/ss-receiving-code-review
git rm -r skills/ss-writing-skills
git rm -r skills/ss-dispatching-parallel-agents
git rm -r skills/ss-executing-plans
git rm -r skills/ss-requesting-code-review
```

- [ ] **Step 2: Verify only 5 skill directories remain**

```bash
ls skills/
```

Expected: `ss-archive`, `ss-brainstorming`, `ss-subagent-driven-development`, `ss-using-superspec`, `ss-writing-plans`

- [ ] **Step 3: Commit**

```bash
git commit -m "refactor: remove 8 skill directories (6 general-purpose, 2 merged)"
```

---

### Task 4: Clean up cross-references in remaining skills

**Files:**
- Modify: `skills/ss-subagent-driven-development/SKILL.md`
- Modify: `skills/ss-archive/SKILL.md`
- Modify: `skills/ss-writing-plans/SKILL.md`
- Modify: `skills/ss-using-superspec/SKILL.md`
- Modify: `skills/ss-using-superspec/references/gemini-tools.md`
- Modify: `skills/ss-using-superspec/references/codex-tools.md`

- [ ] **Step 1: Clean ss-subagent-driven-development/SKILL.md**

In the `## Integration` section:
- Remove: `- **ss-test-driven-development** - Subagents follow TDD for each task`
- (ss-requesting-code-review and ss-executing-plans references already removed in Tasks 1-2)

- [ ] **Step 2: Clean ss-archive/SKILL.md**

In the `## Integration` section, update:
```
- `ss-subagent-driven-development` and `ss-executing-plans` suggest this after all tasks complete
```
To:
```
- `ss-subagent-driven-development` suggests this after all tasks complete
```

- [ ] **Step 3: Clean ss-writing-plans/SKILL.md**

Update the plan document header template — replace:
```
> **For agentic workers:** REQUIRED SUB-SKILL: Use ss-subagent-driven-development (recommended) or ss-executing-plans to implement this plan task-by-task.
```
With:
```
> **For agentic workers:** REQUIRED SUB-SKILL: Use ss-subagent-driven-development to implement this plan task-by-task.
```

Update the Execution Handoff section — replace the two options with a single announcement:
```markdown
## Execution Handoff

After saving the plan and tasks.md:

- Announce: "Plan complete and saved to `changes/YYYY-MM-DD-<topic>/plan.md`. Tasks tracked in `tasks.md`."
- Then use AskUserQuestion with these options:
  - **Subagent-Driven (Recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration
  - **Inline Mode** — Execute tasks directly in this session, batch execution with checkpoints

**REQUIRED SUB-SKILL:** Use ss-subagent-driven-development (subagent mode or inline mode)
```

Remove the separate "If Inline Execution chosen" block that references ss-executing-plans.

- [ ] **Step 4: Clean ss-using-superspec/SKILL.md**

Replace the Skill Priority section to remove references to deleted skills:
```markdown
## Skill Priority

When multiple skills could apply, use this order:

1. **Process skills first** (ss-brainstorming) — determines HOW to approach the task
2. **Implementation skills second** (ss-subagent-driven-development) — guides execution

"Let's build X" → ss-brainstorming first, then implementation skills.
```

Remove the Skill Types section entirely (it references "TDD, debugging" as rigid skills, which no longer exist in the plugin). The remaining skills don't need a rigid/flexible categorization.

- [ ] **Step 5: Clean ss-using-superspec/references/gemini-tools.md**

Update the line referencing ss-executing-plans and ss-dispatching-parallel-agents to reflect the simplified skill set.

- [ ] **Step 6: Clean ss-using-superspec/references/codex-tools.md**

Update the line referencing ss-dispatching-parallel-agents and ss-subagent-driven-development.

- [ ] **Step 7: Verify no dangling references**

```bash
grep -r "ss-executing-plans\|ss-requesting-code-review\|ss-test-driven-development\|ss-systematic-debugging\|ss-verification-before-completion\|ss-receiving-code-review\|ss-dispatching-parallel-agents\|ss-writing-skills" skills/ agents/
```

Expected: no matches (except possibly in the ss-brainstorming SKILL.md which was loaded from plugin cache, not local).

- [ ] **Step 8: Commit**

```bash
git add skills/ agents/
git commit -m "refactor: clean up cross-references to removed skills"
```

---

### Task 5: Update README.md and bump version

**Files:**
- Modify: `README.md`
- Modify: `.claude-plugin/plugin.json`

- [ ] **Step 1: Update README.md**

Update the skill table to show only the 5 remaining skills. Update the workflow description. Remove references to deleted skills. Keep the flow diagram accurate.

The README should reflect:
- 5 skills: ss-brainstorming, ss-writing-plans, ss-subagent-driven-development, ss-archive, ss-using-superspec
- 2 agents: code-reviewer, code-simplifier
- The pipeline: brainstorm → plan → execute (subagent or inline) → archive

- [ ] **Step 2: Bump version**

In `.claude-plugin/plugin.json`, change version from `"0.2.2"` to `"0.3.0"` (significant reduction in plugin surface area).

- [ ] **Step 3: Commit**

```bash
git add README.md .claude-plugin/plugin.json
git commit -m "chore: update README and bump version to 0.3.0"
```
