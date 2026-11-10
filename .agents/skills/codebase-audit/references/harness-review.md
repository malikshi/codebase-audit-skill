# Phase 4 — agent configuration review

Review every agent surface in the repo: `AGENTS.md` and root or directory rule files, skills, workflows and commands, subagent and role definitions, lifecycle hooks, sandbox and permission settings, and MCP or tool-server configs.

## Context and attention economy

- Always-on root instructions (`AGENTS.md`, global system rules) stay lean (<200 lines) and hold only universal repo facts, build/test commands, and global negative constraints.
- Flag task-specific playbooks, verbose references, and domain rules bloating always-on context; they belong in on-demand skills, lazy-loaded workflows, or path-scoped rules.
- Identify duplicate, stale, or contradictory instructions across agent config files.

## Deterministic vs. advisory guardrails

- Find critical rules written only as prose (secret protection, forbidden directories, formatting, mandatory test/lint) that must hold 100% of the time.
- Propose deterministic enforcement — pre/post lifecycle hooks, git hooks, CI or linter gates — to replace fragile prompt-only guards.

## Least privilege and tool hygiene

- Check subagent and role definitions for bloated prompts, overlapping triggers, and overly broad tool or shell access on read-only or research tasks.
- Audit command allowlists and denylists, sandbox permissions, and MCP configs for over-permissioned write/delete/network access, unpinned external packages, and excessive tool-schema token overhead.

## Example backlog row

| Priority | Phase | Location | Issue | Impact | Recommended action |
| :--- | :--- | :--- | :--- | :--- | :--- |
| P0 | 4 | `.commandcode/skills/deploy/SKILL.md:4` | Skill grants `Bash(*)` for a read-only review task | Sec | Restrict to `Bash(git diff:*)` and `Read`; requires owner approval |
