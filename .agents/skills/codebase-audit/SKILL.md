---
name: codebase-audit
description: Run a production-safe four-phase audit of a codebase — delete dead code, fix verified bugs, expose implicit state machines as diagrams, and review agent/harness configuration. Use when the user asks to audit a repo, run a full codebase health check, find dead code, or review agent configuration, skills, and hooks.
---

# Codebase Audit

Production-safe: the repo serves live users. Do not guess on ambiguous behavior, and never break working functionality. Spend effort in this order: **Delete > Simplify > Optimize > Automate**.

## Done means

- The project's test suite, typechecker, and linter pass after every applied change.
- Each applied fix is one independently revertible commit.
- Every item not applied is logged with `file:line`, impact, and a recommended action.

Use the project's own check commands as verification. If none exist, say so and use the smallest deterministic check available (build, syntax, targeted grep).

## Pre-flight

1. `git status` must be clean; record the starting SHA. Stop and ask if the tree is dirty.
2. Detect and run the project test suite, typechecker, and linter. Record the baseline result; the final report compares post-audit status against it.

## Rules for every phase

1. **Verify every change.** Re-run the relevant checks after each edit; revert immediately on failure.
2. **One atomic commit per safe fix.** Apply only changes you are 100% sure are deterministic and behavior-preserving, one at a time, as `audit(<phase>): <what changed> — <why>`. Never batch unrelated fixes. If the repository requires commit approval, prepare the commit and ask before running it.
3. **Review gate for high risk.** Never edit architecture, core business logic, public APIs or schemas, cross-cutting state, or security/permission configuration. Log those in the review backlog instead.

## Phase 1 — Simplify and cut

Find what should not exist: dead code, unused exports, orphan files, redundant abstractions, duplicate utilities, unused dependencies; over-engineered wrappers, premature generalization, unneeded indirection; defensive checks for impossible conditions and legacy workarounds.

Apply only 100% verifiable dead-code removal and isolated behavior-preserving simplifications (`audit(simplify): …`). Hold anything touching live behavior, external contracts, or a contested assumption.

## Phase 2 — Health check

Read the rest and classify:
- **Broken** — latent bugs, unhandled errors or promises, broken imports, off-by-one, null hazards, races, leaks.
- **Inconsistent** — naming drift, mixed error patterns, divergent conventions, mismatched types and schemas.
- **Inefficient** — redundant work, N+1 queries, blocking I/O on hot paths, wrong data structures.
- **Obsolete** — stale flags, deprecated APIs, docs contradicting code, leftover migration shims.

Fix only what you are certain about (`audit(health): …`); list the rest with `file:line` and a concrete recommendation.

## Phase 3 — Surface implicit state machines

Look for tangled boolean flags (`isLoading`/`isError`/`isSuccess`), scattered status strings, and deeply nested transitions. Judge each by UX (impossible or stuck states), DX (self-documenting, type-safe states), and AX (deterministic, local flows agents can modify safely).

For each one:
1. **Visualize first** — draw the current machine as a Mermaid `stateDiagram-v2`, marking dead-end, unhandled, and impossible states.
2. Rank candidates by combined UX/DX/AX impact.
3. Refactor only isolated, self-contained machines whose behavior is 100% preserved, into an explicit machine (discriminated union, transition table, or reducer) as `audit(state): …`. For cross-cutting flows, ship the diagram and proposed design to the backlog with no code change.

## Phase 4 — Agent configuration and harness

Review the repo's agent surfaces against current frontier-model practice: always-on root rules, skills, workflows and commands, subagents, hooks, sandbox and permissions, MCP servers.

Apply only safe, non-destructive cleanups — deduplicate prose, fix broken doc paths, tighten trigger descriptions (`audit(harness): …`). Never auto-modify permissions, hooks, subagent tool scopes, or MCP configs; backlog them for approval.

## Final report

Write it in the exact structure of `references/report.md`.

## Cases → where to look

- Writing the final report → `references/report.md`
- Phase 4 checklist → `references/harness-review.md`
- Dirty tree or failing baseline → stop and ask (Pre-flight)
- Unsure a change is safe → Review gate (rule 3)
