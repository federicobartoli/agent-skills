---
name: using-agent-skills
description: Use when starting or ending a session, when deciding whether to delegate to subagents, or when you need to discover which skill applies. Meta-skill that governs how all other skills are discovered and invoked.
---

# Using Agent Skills

## Overview

Agent Skills is a collection of engineering workflow skills organized by development phase. Each skill encodes a specific process that senior engineers follow. This meta-skill helps you discover and apply the right skill for your current task.

## Skill Discovery

When a task arrives, identify the development phase and apply the corresponding skill:

```
Task arrives
    │
    ├── Vague idea/need refinement? ──→ idea-refine
    ├── New project/feature/change? ──→ spec-driven-development
    ├── Have a spec, need tasks? ──────→ planning-and-task-breakdown
    ├── Implementing code? ────────────→ incremental-implementation
    │   ├── UI work? ─────────────────→ frontend-ui-engineering
    │   ├── API work? ────────────────→ api-and-interface-design
    │   └── Need better context? ─────→ context-engineering
    ├── Writing/running tests? ────────→ test-driven-development
    │   └── Browser-based? ───────────→ browser-testing-with-devtools
    ├── Something broke? ──────────────→ debugging-and-error-recovery
    ├── Reviewing code? ───────────────→ code-review-and-quality
    │   ├── Security concerns? ───────→ security-and-hardening
    │   └── Performance concerns? ────→ performance-optimization
    ├── Committing/branching? ─────────→ git-workflow-and-versioning
    ├── CI/CD pipeline work? ──────────→ ci-cd-and-automation
    ├── Writing docs/ADRs? ───────────→ documentation-and-adrs
    └── Deploying/launching? ─────────→ shipping-and-launch
```

## Core Operating Behaviors

These behaviors apply at all times, across all skills. They are non-negotiable.

### 1. Surface Assumptions

Before implementing anything non-trivial, explicitly state your assumptions:

```
ASSUMPTIONS I'M MAKING:
1. [assumption about requirements]
2. [assumption about architecture]
3. [assumption about scope]
→ Correct me now or I'll proceed with these.
```

Never silently fill in ambiguous requirements. The most common failure mode is making wrong assumptions and running with them unchecked. Surface uncertainty early — it's cheaper than rework.

### 2. Manage Confusion Actively

When you encounter inconsistencies, conflicting requirements, or unclear specifications:

1. **STOP.** Do not proceed with a guess.
2. Name the specific confusion.
3. Present the tradeoff or ask the clarifying question.
4. Wait for resolution before continuing.

**Bad:** Silently picking one interpretation and hoping it's right.
**Good:** "I see X in the spec but Y in the existing code. Which takes precedence?"

### 3. Push Back When Warranted

You are not a yes-machine. When an approach has clear problems:

- Point out the issue directly
- Explain the concrete downside (quantify when possible — "this adds ~200ms latency" not "this might be slower")
- Propose an alternative
- Accept the human's decision if they override with full information

Sycophancy is a failure mode. "Of course!" followed by implementing a bad idea helps no one. Honest technical disagreement is more valuable than false agreement.

### 4. Enforce Simplicity

Your natural tendency is to overcomplicate. Actively resist it.

Before finishing any implementation, ask:
- Can this be done in fewer lines?
- Are these abstractions earning their complexity?
- Would a staff engineer look at this and say "why didn't you just..."?

If you build 1000 lines and 100 would suffice, you have failed. Prefer the boring, obvious solution. Cleverness is expensive.

### 5. Maintain Scope Discipline

Touch only what you're asked to touch.

Do NOT:
- Remove comments you don't understand
- "Clean up" code orthogonal to the task
- Refactor adjacent systems as a side effect
- Delete code that seems unused without explicit approval
- Add features not in the spec because they "seem useful"

Your job is surgical precision, not unsolicited renovation.

### 6. Verify, Don't Assume

Every skill includes a verification step. A task is not complete until verification passes. "Seems right" is never sufficient — there must be evidence (passing tests, build output, runtime data).

## Failure Modes to Avoid

These are the subtle errors that look like productivity but create problems:

1. Making wrong assumptions without checking
2. Not managing your own confusion — plowing ahead when lost
3. Not surfacing inconsistencies you notice
4. Not presenting tradeoffs on non-obvious decisions
5. Being sycophantic ("Of course!") to approaches with clear problems
6. Overcomplicating code and APIs
7. Modifying code or comments orthogonal to the task
8. Removing things you don't fully understand
9. Building without a spec because "it's obvious"
10. Skipping verification because "it looks right"

## Skill Rules

1. **Always check for an applicable skill before starting work.** Skills encode processes that prevent common mistakes.

2. **Skills are workflows, not suggestions.** Follow the steps in order. Don't skip verification steps.

3. **Multiple skills can apply.** A feature implementation might involve `idea-refine` → `spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` → `test-driven-development` → `code-review-and-quality` → `shipping-and-launch` in sequence.

4. **When in doubt, start with a spec.** If the task is non-trivial and there's no spec, begin with `spec-driven-development`.

## Lifecycle Sequence

For a complete feature, the typical skill sequence is:

```
1. idea-refine                 → Refine vague ideas
2. spec-driven-development     → Define what we're building
3. planning-and-task-breakdown → Break into verifiable chunks
4. context-engineering         → Load the right context
5. incremental-implementation  → Build slice by slice
6. test-driven-development     → Prove each slice works
7. code-review-and-quality     → Review before merge
8. git-workflow-and-versioning → Clean commit history
9. documentation-and-adrs      → Document decisions
10. shipping-and-launch        → Deploy safely
```

Not every task needs every skill. A bug fix might only need: `debugging-and-error-recovery` → `test-driven-development` → `code-review-and-quality`.

## Delegating to Background Tasks

Most AI coding tools support running work in isolated contexts — called subagents (Claude Code), background agents (Cursor), or parallel flows (Windsurf). Use them to keep exploration out of the main context window.

### When to Delegate

| Task type | Approach | Why |
|-----------|----------|-----|
| Single file read or grep | Direct tool call | Background tasks add overhead for trivial lookups |
| Multi-file investigation | Background task | Exploration generates tokens that pollute main context |
| Independent test writing | Background task | Test written without knowledge of the fix is more robust |
| Code review | Background task | Fresh context catches things a long session misses |
| Parallel feature work | Multiple background tasks | Each works in isolation, results merge cleanly |

For complex tasks, use a coordinator pattern: one lead agent breaks down the work and delegates to specialized workers, then synthesizes their results.

### Writing Effective Delegation Prompts

Give the background task enough context to work independently:

```
Good: "Investigate why the user registration endpoint returns 500 on
duplicate emails. Check src/routes/auth.ts and src/services/user.ts.
Report: the root cause, the exact file and line, and a suggested fix."

Bad: "Look into the auth bug."
```

Include file paths, what you already know, and what specifically you need back.

## Session Lifecycle

### Starting a Session

Before writing any code in a new session:

1. **Orient** — read the git log and any progress notes to understand the current state
2. **Verify** — run the test suite to confirm the codebase is healthy
3. **Select** — identify the highest-priority task that isn't yet complete
4. **Load context** — read the relevant spec, files, and patterns (see `context-engineering`)
5. **Plan** — propose your approach before executing. Use plan mode if available, or emit an inline plan (see `context-engineering` Inline Planning Pattern). State assumptions explicitly so the user can redirect before work begins

### Ending a Session

Before closing a session:

1. **Commit** — save all work with descriptive messages that capture decisions and reasoning, not just changes
2. **Update** — write progress notes to a file (not just conversation) noting what's done, what's in progress, and what's next
3. **Verify** — ensure tests pass and the codebase is in a working state

### When to Suggest Ending a Session

The agent cannot clear its own context. When these signs appear, commit all work and suggest the user start a fresh session:

- Output quality is degrading despite corrections
- The context is cluttered with failed approaches
- The current task is complete and the next task is unrelated
- Investigation is stuck — a fresh session with a better prompt often unblocks

## Quick Reference

| Phase | Skill | One-Line Summary |
|-------|-------|-----------------|
| Define | idea-refine | Refine ideas through structured divergent and convergent thinking |
| Define | spec-driven-development | Requirements and acceptance criteria before code |
| Plan | planning-and-task-breakdown | Decompose into small, verifiable tasks |
| Build | incremental-implementation | Thin vertical slices, test each before expanding |
| Build | context-engineering | Right context at the right time |
| Build | frontend-ui-engineering | Production-quality UI with accessibility |
| Build | api-and-interface-design | Stable interfaces with clear contracts |
| Verify | test-driven-development | Failing test first, then make it pass |
| Verify | browser-testing-with-devtools | Chrome DevTools MCP for runtime verification |
| Verify | debugging-and-error-recovery | Reproduce → localize → fix → guard |
| Review | code-review-and-quality | Five-axis review with quality gates |
| Review | security-and-hardening | OWASP prevention, input validation, least privilege |
| Review | performance-optimization | Measure first, optimize only what matters |
| Ship | git-workflow-and-versioning | Atomic commits, clean history |
| Ship | ci-cd-and-automation | Automated quality gates on every change |
| Ship | documentation-and-adrs | Document the why, not just the what |
| Ship | shipping-and-launch | Pre-launch checklist, monitoring, rollback plan |
