# Superspec

Superspec is a development workflow plugin for Claude Code that combines the best of [superpowers](https://github.com/obra/superpowers) (workflow quality, TDD, subagent orchestration) with structured change management and architectural knowledge capture.

## How it works

When you start building something, Superspec doesn't jump into code. It walks you through a design conversation, writes a spec, creates an implementation plan with persistent task tracking, and then executes it with subagent-driven development and two-stage code review.

What's different from superpowers:

- **Per-change directory** — Each feature gets a `changes/YYYY-MM-DD-<topic>/` directory with `design.md`, `plan.md`, and `tasks.md`. Everything lives together.
- **Persistent task tracking** — `tasks.md` survives across conversations. Resume where you left off instead of starting over.
- **Archive skill** — When you're done, capture the architectural knowledge (why it was built, key decisions, alternatives considered) as a standalone artifact. Not file copies — distilled knowledge.
- **No CLI dependencies** — Pure file operations. No external tools required.

## Installation

### Claude Code (GitHub)

```bash
/plugin marketplace add CoyoteLeo/superspec
/plugin install superspec@superspec
```

### Claude Code (local development)

```bash
claude --plugin-dir /path/to/superspec
```

### Verify Installation

Start a new session and try building something. Superspec skills trigger automatically. Or invoke directly:

```
/superspec:ss-brainstorming
```

All skills use the `ss-` prefix for easy identification:

```
/superspec:ss-brainstorming
/superspec:ss-writing-plans
/superspec:ss-archive
```

## The Workflow

```
ss-brainstorming → changes/YYYY-MM-DD-<topic>/design.md
       ↓
ss-writing-plans → plan.md + tasks.md
       ↓
execute (ss-subagent-driven-development or ss-executing-plans) → tasks.md checkbox tracking
       ↓
ss-archive → changes/archive/YYYY-MM-DD-<topic>.md (knowledge artifact)
```

1. **ss-brainstorming** — Refines ideas through questions, explores alternatives, presents design in sections. Saves to `changes/YYYY-MM-DD-<topic>/design.md`.

2. **ss-writing-plans** — Breaks work into bite-sized tasks (2-5 min each) with exact file paths, complete code, and verification steps. Generates `plan.md` and `tasks.md` in the change directory.

3. **ss-subagent-driven-development** or **ss-executing-plans** — Fresh subagent per task with two-stage review (spec compliance, then code quality). Tracks progress in both TodoWrite (in-session) and `tasks.md` (persistent). Git operations are left to the user.

4. **ss-archive** — Synthesizes design.md + plan.md + tasks.md into an architectural knowledge artifact capturing purpose, decisions, alternatives, scope, and lessons learned.

## What's Inside

### Core Pipeline

| Skill | Purpose |
|-------|---------|
| **ss-brainstorming** | Socratic design refinement → design.md |
| **ss-writing-plans** | Implementation plans → plan.md + tasks.md |
| **ss-subagent-driven-development** | Per-task subagent execution with two-stage review |
| **ss-executing-plans** | Batch execution with checkpoints (alternative) |
| **ss-archive** | Architectural knowledge artifact generation |

### Supporting Skills

| Skill | Purpose |
|-------|---------|
| **ss-test-driven-development** | RED-GREEN-REFACTOR cycle |
| **ss-systematic-debugging** | 4-phase root cause process |
| **ss-verification-before-completion** | Verify before claiming success |
| **ss-requesting-code-review** | Dispatch code reviewer subagent |
| **ss-receiving-code-review** | Respond to feedback with rigor |
| **ss-dispatching-parallel-agents** | Concurrent subagent workflows |
| **ss-writing-skills** | Create new skills following best practices |
| **ss-using-superpowers** | Introduction to the skills system |

### Agents

| Agent | Purpose |
|-------|---------|
| **ss-code-reviewer** | Automated code review subagent |
| **code-simplifier** | Code clarity and maintainability review |

## Change Directory Structure

```
changes/
  2025-03-25-auth-refactor/     # Active change
    design.md                    # From ss-brainstorming
    plan.md                      # From ss-writing-plans
    tasks.md                     # Persistent task tracking
  archive/
    2025-03-25-auth-refactor.md  # Knowledge artifact (not file copies)
```

## Key Differences from Superpowers

| Aspect | Superpowers | Superspec |
|--------|------------|-----------|
| **State** | Stateless handoffs, TodoWrite only | Per-change directory + persistent tasks.md |
| **File layout** | `docs/superpowers/specs/` and `plans/` | `changes/YYYY-MM-DD-<topic>/` (co-located) |
| **Cross-session** | Start over each conversation | Resume from tasks.md |
| **Knowledge capture** | None | Archive skill → architectural knowledge artifacts |
| **CLI dependencies** | None | None |
| **Git operations** | Agent executes directly | User handles git workflow |

## Philosophy

- **Test-Driven Development** — Write tests first, always
- **Persistent over ephemeral** — Track state across conversations
- **Knowledge over files** — Archive captures why, not what
- **Systematic over ad-hoc** — Process over guessing

## Contributing

1. Fork the repository
2. Create a branch for your skill
3. Follow the `ss-writing-skills` skill for creating and testing new skills
4. Submit a PR

## License

MIT License

## Credits

Superspec builds on the foundation of [superpowers](https://github.com/obra/superpowers) by [Jesse Vincent](https://blog.fsck.com). The core workflow skills (brainstorming, TDD, subagent-driven development, code review) originate from that project. Superspec adds structured change management, persistent task tracking, and architectural knowledge archiving.
