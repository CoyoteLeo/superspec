---
name: ss-deriving-spec-from-pm-pr
description: Use when a non-engineer (PM / designer / product owner) has opened a frontend PR sketching a new feature's UI and user-flow with mocked data or stubbed API calls, and engineering needs to extract requirements, reason about backend / API support, and produce an OpenSpec change directory ready for ss-writing-plans
---

# Deriving Spec From a Frontend Draft PR

## Overview

Product owners often communicate new feature ideas by opening a frontend PR with mocked data, draft layouts, or stubbed API calls. The PR shows what the user sees — but the backend does not yet exist, or only partially exists. This skill turns that PR into an OpenSpec change directory (`openspec/changes/YYYY-MM-DD-<topic>/`) ready for `ss-writing-plans` to consume.

**Announce at start:** "I'm using the ss-deriving-spec-from-pm-pr skill to turn the PR into a spec."

**Core principle:** **The product-owner-decided UI is input, not a discussion topic.** Layout, copy, and user-flow as drafted in the PR are constraints. What gets re-thought is the backend, API contract, data model, validation, error handling, permissions, and cross-service integration.

<HARD-GATE>
Do NOT redesign the PR's UI. Do NOT auto-invoke `ss-writing-plans`. Do NOT write implementation steps. This skill ends at producing the change directory; the user decides when to invoke `ss-writing-plans`.
</HARD-GATE>

## When to Use

- A non-engineer has opened a PR (often draft) in a frontend repo with mocked data or stub API calls
- UI / user-flow is drafted but the backend doesn't exist yet, or only partially exists
- Engineering needs to convert the PR into a real, executable spec without overriding the UI decisions

**Do NOT use when:** The author shared only a Figma link, Slack thread, or text idea without a PR — use `ss-brainstorming` instead.

## Checklist

You MUST create a TodoWrite task for each stage and complete in order. Each stage requires explicit user confirmation before advancing.

1. **Stage 1 — Ingest the PR**
2. **Stage 2 — Reverse-engineer the user flow**
3. **Stage 3 — Identify backend gaps**
4. **Stage 4 — Discuss & clarify** (mandatory; uses AskUserQuestion)
5. **Stage 5 — Generate the OpenSpec change directory**
6. **Handoff** — tell user to invoke `ss-writing-plans`

## Stage 1: Ingest the PR

Ask the user for the PR URL or `<owner>/<repo>#<number>`. Then:

```bash
gh pr view <pr> --json title,body,files,comments,headRefName,baseRefName,url,author
gh pr diff <pr>
```

Summarize for the user:

- Title, author, target repo, base / head branches
- Files changed, grouped by purpose: UI components, routes, API clients, fixtures / mocks, tests, i18n
- PR body and meaningful reviewer comments
- Whether response shapes are real or mocked (look for fixtures, MSW handlers, hard-coded JSON, dev-only flags)

Confirm summary with user before proceeding.

## Stage 2: Reverse-engineer the user flow

From the diff:

- List every new / modified component, route, form, store / atom
- Trace user actions: which control → which handler → which state change → which API call → which screen transition
- Output **(a)** a numbered user-flow list, and **(b)** a Mermaid `sequenceDiagram` showing User → Frontend → Backend
- Mark every "PR-fixed" UI element explicitly (these are constraints, not negotiable)

Confirm the flow with the user. If the user corrects your reading of the UI, update — but never propose UI changes yourself.

## Stage 3: Identify backend gaps

For each API call observed in the PR (e.g. `fetch`, `axios`, `SWR`, RTK Query, project-specific API clients):

| Endpoint | Method | Request shape | Response shape (mock?) | Auth / scoping | Pagination | Error cases | Status |

Status column values:

- **EXISTS** — endpoint exists in backend, no change
- **PARTIAL** — endpoint exists but missing fields / auth / edge cases
- **MISSING** — must build new

Cross-reference the corresponding backend repo. Use `Grep` / `Read` against the backend service to verify (do NOT guess). If the backend follows a layered architecture (e.g., migration / domain / adapter / service / router, or any equivalent split), call out which layers each gap touches.

Reminder: when the work touches multiple services or both FE + BE, recommend splitting into multiple PRs along service seams. Note the recommended split in `proposal.md`.

## Stage 4: Discuss & clarify (mandatory)

Use `AskUserQuestion` to resolve ambiguities. Topics:

- **Business rules** — validation, limits, defaults
- **Edge cases** — empty / loading / error / race conditions
- **Permissions** — who can call this? scope (org / tenant / role)? feature-flagged?
- **Cross-service impact** — telemetry events, API gateway / routing changes, integrations with upstream systems, feature flags
- **Data lifecycle** — soft-delete vs hard-delete, cascade semantics, retention. Defer to project conventions on deletion safety; flag any cascade-delete proposal explicitly so it gets reviewed.

Do **NOT** ask the user about UI / layout / copy. If you catch yourself drafting a UI question, STOP — that's the product owner's call.

## Stage 5: Generate the OpenSpec change directory

**Where to write:**

1. If the FE repo has its own `openspec/` directory, use it
2. Else if the backend counterpart repo has `openspec/`, use it
3. Else ask the user

**Directory:** `<repo>/openspec/changes/YYYY-MM-DD-<topic>/`

**Files:**

- `proposal.md` — scope, capabilities, impact. **Must include:** "Frontend draft: PR <url>" and the recommended PR-split plan.
- `design.md` — architecture, decisions, rationale.
  - UI / Layout section **references PR component paths** (e.g., `frontend/src/pages/<Page>.tsx`); do NOT redraw or restyle.
  - Backend section is the meat: data model, API contracts, layered design (per the project's architecture), cross-service touchpoints.
  - **Decisions table:** what we kept from the PR, what we changed (and why), what's new.
- `specs/` — one file per capability:
  - API contract (path, method, request, response, errors, auth)
  - Data model (follow the project's conventions for IDs, types, nullability, and deletion semantics)
  - Business rules
- `tasks.md` — high-level checklist; one line per major task. `ss-writing-plans` will expand each into bite-sized steps:

```markdown
# Tasks: <Feature Name>
- [ ] Migration: <table changes>
- [ ] Domain / Models: <pure logic, types>
- [ ] Adapter / Repository: <DB, external API>
- [ ] Service: <orchestration>
- [ ] API / Router: <endpoints>
- [ ] Frontend: wire real API into PR #<num>
```

(Adjust task names to match the project's architectural vocabulary.)

After writing, scan once: does `design.md` exist? does `tasks.md` use `- [ ]`? does `proposal.md` link the PR? If yes, proceed to handoff.

## Handoff

State exactly:

> "Spec ready at `<repo>/openspec/changes/YYYY-MM-DD-<topic>/design.md`.
> Backend gaps documented in `specs/`.
> **Next step:** invoke `ss-writing-plans` to produce the implementation plan. I will not invoke it automatically."

Stop. Wait for user.

## Common Mistakes

| ❌ Don't | ✅ Do |
|---------|------|
| Redesign the PR's UI | Treat UI / layout / copy as constraint; reference PR component paths |
| Trust the PR's mock API as a backend contract | Challenge every mock; design the real contract in Stage 3 |
| Skip Stage 4 (clarify) | Always run AskUserQuestion before writing `design.md` |
| Write step-by-step implementation steps | That's `ss-writing-plans`'s job |
| Auto-invoke `ss-writing-plans` | Hand off via text; let the user invoke |
| Modify `ss-writing-plans` or `ss-brainstorming` | Cross-reference only |
| Bundle FE + BE + migration into one task list | Split along service seams; migrations are typically their own PR |

## Red Flags — STOP

- "The PR's UI seems off, let me redesign it" → STOP. Surface to the author in a PR comment; don't change the spec.
- "I'll just trust the mock response shape" → STOP. Challenge it in Stage 3.
- "I'll skip the discussion stage and write design.md" → STOP. Stage 4 is mandatory.
- "Let me also start writing the implementation plan" → STOP. That's `ss-writing-plans`.
- "Let me also kick off `ss-writing-plans` for them" → STOP. Hand off via text only.

## Cross-references (read-only)

- **REQUIRED HANDOFF SKILL:** `ss-writing-plans` (next step after this skill ends)
- **SIBLING SKILL (different input):** `ss-brainstorming` — use that when there is no PR
- **META:** `ss-writing-skills` — TDD-for-skills used to author this skill
