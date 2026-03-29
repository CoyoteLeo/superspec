---
name: ss-archive
description: Archive a completed change by generating an architectural knowledge artifact. Use when implementation is complete and you want to capture the design intent, decisions, and target scenarios for future reference.
---

# Archive Change

Generate an architectural knowledge artifact from a completed change. This is NOT about copying files — it's about distilling the essential knowledge that future conversations need to understand why this was built and how it works.

**Announce at start:** "I'm using the ss-archive skill to capture the architectural knowledge from this change."

## Input

Optionally specify a change directory path (e.g., `changes/2025-03-25-auth-refactor/`). If omitted, look for active change directories under `changes/` and use the AskUserQuestion tool to let the user select one.

## The Process

### Step 1: Locate the Change

1. If path provided, use it
2. Otherwise, scan `changes/` for directories (excluding `archive/`)
3. If multiple found, use the AskUserQuestion tool to let user select
4. If none found, abort with message

### Step 2: Read Change Artifacts

Read all available artifacts from the change directory:
- `design.md` — architectural design and spec
- `plan.md` — implementation plan
- `tasks.md` — task completion status

Not all artifacts need to exist. Work with what's available.

### Step 3: Generate Architectural Knowledge Artifact

Synthesize the artifacts into a single knowledge document. This document answers the questions a future developer (or AI agent) would ask:

**Structure:**

```markdown
# [Feature/Change Name]

**Date:** YYYY-MM-DD
**Status:** Completed | Partial (N/M tasks)
**Change dir:** changes/YYYY-MM-DD-<topic>/

## Purpose & Target Scenarios

Why was this built? What problem does it solve? Who benefits?
Describe the specific scenarios this enables — concrete, not abstract.

## Architecture & Key Decisions

High-level architecture: what components exist, how they interact, what patterns are used.

For each significant decision, capture:
- **Decision:** What was chosen
- **Alternatives considered:** What was rejected and why
- **Rationale:** Why this approach won

## Scope & Boundaries

What this change does and does NOT cover. What was intentionally deferred or excluded.

## Integration Points

How this connects to the rest of the system. What depends on it, what it depends on.

## Lessons & Notes

Anything surprising, non-obvious, or worth remembering for future work in this area.
```

**Guidelines:**
- Write for someone with zero context about this change
- Prioritize WHY over WHAT — the code shows what, the archive explains why
- Keep it concise — aim for 200-500 words total, not a novel
- Include alternatives considered and rejected (from design.md)
- Note anything that was surprising or non-obvious during implementation
- If tasks.md shows incomplete tasks, note what was deferred and why

### Step 4: Save to Archive

1. Create directory: `mkdir -p changes/archive/`
2. Save to: `changes/archive/YYYY-MM-DD-<topic>.md`
3. If file already exists, ask user whether to overwrite or rename

### Step 5: Report

```
## Archive Complete

**Change:** <topic>
**Archived to:** changes/archive/YYYY-MM-DD-<topic>.md
**Tasks:** N/M complete
**Source:** changes/YYYY-MM-DD-<topic>/

The architectural knowledge has been captured. The source change directory is preserved — delete it manually when ready.
```

## Guardrails

- Never delete the source change directory automatically — user decides when to clean up
- No CLI dependencies — pure file operations
- Work with partial artifacts (not all files need to exist)
- Ask before overwriting existing archives
- Focus on architectural knowledge, not implementation details
- The archive is a knowledge artifact, not a backup

## Integration

**Called by:**
- `ss-subagent-driven-development` and `ss-executing-plans` suggest this after all tasks complete
- User directly via `/superspec:ss-archive`

**Reads from:**
- Change directory artifacts (design.md, plan.md, tasks.md)

**No dependencies on:**
- External CLIs (no openspec commands)
- Specific schema formats
