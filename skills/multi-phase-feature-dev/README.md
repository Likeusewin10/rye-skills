<div align="right">
  <a href="README.zh-CN.md">中文</a> | <strong>English</strong>
</div>

# multi-phase-feature-dev

> Orchestrator pattern for complex, multi-phase feature development with parallel subagents.

[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-blue)](https://github.com/Likeusewin10/rye-skills)
[![Requires ECC](https://img.shields.io/badge/requires-ECC-orange)](https://github.com/affaan-m/ECC)

---

## Dependencies

This skill relies on two components from [Everything Claude Code (ECC)](https://github.com/affaan-m/ECC):

| Component | Type | Used for |
|-----------|------|----------|
| `code-reviewer` | agent | Parallel code review in each phase |
| `coding-standards` | skill | Coding conventions injected into subagent prompts |

Install ECC before using this skill.

---

## Background

After working through multiple long development sessions, I kept running into the same two problems: context window exhaustion mid-task, and agents "forgetting" earlier decisions by the time they reached later phases — writing the back half without remembering what the front half had committed to.

The root cause is that long tasks are fundamentally sequential in practice, even when they don't need to be. Each agent waits for the previous one, context accumulates, and by phase 4 or 5 the agent is operating on a compressed summary of what it decided in phase 1.

This workflow fixes that by making the main agent own the one thing that matters most — the interface contract — and then letting subagents work in parallel from that frozen contract. Each subagent starts fresh with only what it needs. The main agent stays lean because it's orchestrating, not implementing. Code review runs alongside the next phase instead of blocking it.

It's not a framework. It's a simple set of rules I wrote down after noticing the same failure modes repeat across projects.

---

## Install

```bash
mkdir -p ~/.claude/skills
curl -o ~/.claude/skills/multi-phase-feature-dev.md \
  https://raw.githubusercontent.com/Likeusewin10/rye-skills/main/skills/multi-phase-feature-dev/multi-phase-feature-dev.md
```

Invoke in Claude Code:

```
/multi-phase-feature-dev
```

---

## How It Works

```
Main Agent
  │
  ├── Reads codebase → designs interface contract → freezes it
  │
  ├── Launches in parallel:
  │     ├── Backend subagent  (implements against frozen contract)
  │     └── Frontend subagent (implements against frozen contract)
  │
  ├── Both complete → design next phase contract
  │
  ├── Launches in parallel:
  │     ├── Backend subagent  (phase N+1)
  │     ├── Frontend subagent (phase N+1)
  │     └── code-reviewer     (phase N)  ← non-blocking, from ECC
  │
  └── Repeat until done → commit
```

The key insight: the main agent defines the complete API contract (endpoints, request/response shapes, types) **before** any subagent starts. Frontend and backend both implement against the same frozen spec independently. This eliminates the most common failure mode — type mismatches between frontend and backend caused by one agent inferring what the other "probably" returned.

---

## When to Use

| ✅ Use | ❌ Skip |
|--------|---------|
| Feature has frontend + backend split | Single-file change |
| Touches more than 3 files | Pure frontend or pure backend only |
| Single phase estimated > 30 min | Urgent hotfix |

---

## Contents

See [`multi-phase-feature-dev.md`](multi-phase-feature-dev.md) for the full skill, including:

- Main agent rules (interface ownership, parallel execution, review priority)
- Interface contract template
- Backend / frontend / code-reviewer subagent task templates
- Full execution flow
- Common pitfalls

---

← [Back to all skills](../../README.md)
