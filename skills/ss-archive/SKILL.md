---
name: ss-archive
description: Use when a change packet's implementation is complete and you want to promote it into the project's permanent spec layout before closing the change out. Typical trigger phrases include "archive this change", "we're done with change X", or invocation after ss-executing-plans / ss-subagent-driven-development report all tasks complete.
---

# Archive Change

Promote a completed change into the project's permanent knowledge layout: the active change folder moves under `changes/archive/`, and the capability it implemented gets a machine-validatable `spec.md` plus a human-readable `overview.md` under `specs/<capability>/`.

**Announce at start:** "I'm using the ss-archive skill to archive this change and capture its architectural knowledge."

The source change folder is preserved under `changes/archive/` — this is a move, not a delete. The artifact you produce is a distilled companion to the formal spec, not a copy of proposal/design/plan.

## Input

Optionally specify the change identifier (e.g. `2025-03-25-auth-refactor`). If omitted, list active change folders under `changes/` (excluding `archive/`); if multiple, ask the user. If none, abort.

## Process

### Step 1: Read Change Artifacts

From `changes/<change-name>/`:
- `proposal.md` — scope and motivation
- `design.md` — architectural design
- `plan.md` — implementation plan
- `tasks.md` — completion status
- `specs/<capability>/spec.md` — the capability spec

Extract the capability name from the `specs/<capability>/` path. If the change spans multiple capabilities, ask the user which to archive — each capability gets its own overview.

Partial artifacts are fine — work with what's there.

### Step 2: Draft the Overview

Write a single overview document per capability. Target 400–600 words.

**Structure:**

```markdown
# <Capability Name> — Overview

Human-readable companion to [`spec.md`](spec.md). Start here to understand **why** the capability exists and **how** the pieces fit together; read `spec.md` for the exact requirements and scenarios.

## Purpose

Why does this capability exist? What friction did it remove? Describe the concrete user scenarios it enables — not abstract goals.

## Architecture & Key Design

Components, interactions, data flow. Each bolded paragraph captures one significant design choice — what was chosen, written as a statement about the shipped system.

## Scope & Boundaries

What this capability does and does NOT cover. Non-goals called out so future work doesn't accidentally stretch the contract.

## Integration Points

External services, other repos, infra dependencies. Enough detail to trace "if X breaks, what in our system breaks".

## Notes for Future Work

Forward-looking gotchas for anyone extending or operating this system. Frame as "when you touch X, remember Y" — not "we had a bug and fixed it".

---

**Provenance:** link to the archived change folder.
```

**Writing rules (important):**

- **Describe the shipped system, not history.** Never write "Alternatives considered / rejected" sections. Never frame lessons as "we made a mistake in PR #N". Those narratives belong in git history, not the overview.
- **Prioritise WHY over WHAT.** `spec.md` already enumerates WHAT; the overview exists to convey intent.
- **Be concrete.** Use actual file paths, function names, env var names, PR numbers. Vague prose ages badly.
- **No "alternatives rejected" block.** If a past decision is load-bearing, fold it into the architecture prose as a current-state statement ("X is a safety net, not a tenant boundary").

### Step 3: Promote the Spec

Make sure `specs/<capability>/` exists; create it if needed.

Promote the change's spec into `specs/<capability>/spec.md`:

- **New capability** (no existing `specs/<capability>/spec.md`): the change's spec becomes the capability's spec. Copy the content over.
- **Existing capability** (spec.md already there): the change's spec is a delta to the existing one. Merge the requirements into the existing spec, respecting whatever add/modify/remove markers the change's spec uses. If you're unsure how to merge, ask the user before overwriting.

Make sure the final `specs/<capability>/spec.md` has a concrete Purpose section — if it's empty or placeholder text, fill it with 2–3 sentences paraphrased from the overview's Purpose.

### Step 4: Write the Overview

Write the drafted overview to `specs/<capability>/overview.md`. If the file already exists (the capability is being revised, not created), ask the user whether to overwrite or merge — overview.md evolves with the capability.

### Step 5: Move the Source Folder

Move `changes/<change-name>/` to `changes/archive/<YYYY-MM-DD>-<change-name>/` where `<YYYY-MM-DD>` is today's date. This preserves the full decision trail (proposal, design, plan, tasks, spec delta) as a frozen historical record.

Update the Provenance footer in `overview.md` to point at the new archived-folder path.

### Step 6: Report

```
## Archive Complete

**Change:** <change-name> → changes/archive/<YYYY-MM-DD>-<change-name>/
**Spec:** specs/<capability>/spec.md — <N> requirements
**Overview:** specs/<capability>/overview.md
```

## File Roles

| Path | Role | Evolves? |
|------|------|----------|
| `specs/<capability>/spec.md` | Machine-validatable contract | Yes — new changes add/modify requirements |
| `specs/<capability>/overview.md` | Human-readable companion | Yes — updated in place by future archives |
| `changes/archive/<prefixed-name>/` | Frozen source packet (proposal / design / plan / tasks / spec delta) | No — historical |

Future changes touching the same capability should UPDATE `overview.md` rather than create a second one. Don't accumulate per-change archive docs under `specs/`.

## Guardrails

- **No "alternatives rejected" narratives.** Describe shipped state. If a rejected alternative is still load-bearing context, fold it into architecture prose as current-state.
- **No backward-looking lessons framing.** "Notes for Future Work" is forward-looking — "when you do X, remember Y", not "we made mistake Z".
- **Archive = move, not delete.** The source change folder is preserved under `changes/archive/`, not removed. Full history stays accessible.
- **Ask before overwriting.** If `spec.md` or `overview.md` already exists in `specs/<capability>/`, don't silently clobber — ask the user how to handle it.

## Integration

**Called by:**
- `ss-subagent-driven-development` and `ss-executing-plans` suggest this after all tasks complete.
- User directly via `/superspec:ss-archive` or `/ss-archive`.

**Reads from:**
- Active change artifacts under `changes/<change-name>/` (`proposal.md`, `design.md`, `plan.md`, `tasks.md`, `specs/<capability>/spec.md`).

**Writes to:**
- `specs/<capability>/spec.md` — promoted from the change's spec (created or merged).
- `specs/<capability>/overview.md` — newly drafted digest.
- Moves `changes/<change-name>/` to `changes/archive/<prefixed-name>/`.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing or placeholder Purpose in `specs/<capability>/spec.md` | Fill with 2–3 sentences from the overview. Don't ship with an empty or TBD Purpose. |
| Writing a "Decisions → Alternatives → Rationale" bullet list for every choice | Fold decisions into prose. The overview is a digest, not a decision log. |
| Running the skill before all tasks are complete | Confirm `tasks.md` shows all tasks done (or the user explicitly accepts the gap) first. |
| Creating `overview.md` in a capability folder that already has one | Ask first. The existing overview likely carries context from prior changes that shouldn't be silently overwritten. |
| Deleting the source change folder instead of moving it | Archive is a move to `changes/archive/`, not a delete. The full decision trail must stay accessible. |
