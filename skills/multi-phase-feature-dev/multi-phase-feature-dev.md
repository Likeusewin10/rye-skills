---
name: multi-phase-feature-dev
description: Orchestrator pattern for multi-phase feature development. Main agent defines interfaces, parallel frontend/backend subagents implement against frozen contracts, code review runs non-blocking alongside next phase. Use for features with clear frontend/backend split, 3+ files, and phases > 30 min each.
version: 1.0.0
---

# Multi-Phase Feature Development

Orchestrator pattern for complex features with parallel subagent execution and non-blocking code review.

## Core Idea

Main agent acts as orchestrator:
1. Decompose phases and define interfaces
2. Dispatch tasks to subagents
3. Handle review findings
4. Maintain docs and commits

**Key principle: main agent defines all interfaces. Frontend and backend subagents implement in parallel against the same frozen contract. Code review never blocks progress.**

```
Main Agent
  │
  │ Before phase N
  ├── Main agent reads codebase and defines interface contract (API endpoints + request/response + types)
  ├── Freeze interface
  │
  │ After freeze, launch in parallel
  ├── Backend subagent: implement API routes, DB operations, utilities
  ├── Frontend subagent: implement UI and API calls against frozen contract
  └── (Previous phase) code-reviewer: review phase N-1 quality
  │
  │ After both backend + frontend complete
  ├── code-reviewer: review current phase
  └── Main agent handles findings → start next phase
```

---

## Main Agent Rules

### Rule 1: Main Agent Owns Interface Design

Never delegate interface design to subagents. Before launching any subagent, the main agent reads the codebase, designs the interface, and writes a complete contract.

Interface contract format:

```
## Interface Contract (Phase N)

### New Functions / Utilities
- `functionName(param: Type): ReturnType` — description

### New API Endpoints
- `METHOD /path`
  - Request: `{ field: Type }`
  - Response: `{ field: Type }` (complete JSON structure, no ambiguity)

### New Type Definitions
- `TypeName { field: Type; ... }`

### Backend Subagent Dependencies
- Existing files to read
- Modules to import

### Frontend Subagent Dependencies
- API endpoints to call with complete response structure
- Existing component files to read
```

### Rule 2: Parallel Frontend + Backend

After interface freeze, launch both subagents simultaneously:

- **Backend subagent**: implement API routes, DB queries, utilities — interface contract is the source of truth
- **Frontend subagent**: implement UI components, API calls — interface contract is the source of truth, does not wait for backend

Both subagents work independently from the same contract.

### Rule 3: Code Review Runs in Parallel

```
Phase N frontend + backend complete
  ├── Immediately start: phase N+1 interface design + subagents
  └── Simultaneously start: code-reviewer for phase N
```

Review findings are processed by priority after they arrive — they never block the next phase from starting.

### Rule 4: Finding Priority

| Severity | Action |
|---|---|
| CRITICAL | Interrupt all subagents immediately. Main agent fixes before continuing. |
| HIGH | Reviewer fixes inline with Edit tool, marks "fixed". Does not interrupt next phase. |
| MEDIUM | Log to docs. Address after current phase completes. |
| LOW | Log to docs. Optional. |

### Rule 5: Build Verification

Each subagent runs a build check after completing. Treat build failure as CRITICAL.

Common commands (replace with your project's):
- TypeScript: `npx tsc --noEmit`
- Go: `go build ./...`
- Python: `python -m py_compile src/**/*.py`
- Rust: `cargo check`

### Rule 6: Documentation Sync

- Create implementation doc before starting (`docs/feature-name.md`)
- Update phase status after each phase completes (⏳ → ✅) and record review conclusions
- Commit after all phases complete with a summary covering all phases

---

## Subagent Task Templates

### Backend Subagent

```
## Task: Implement Phase X Backend — [Phase Name]

### Interface Contract (frozen, do not modify)
[Complete interface contract from main agent]

### Existing Files to Read
[List file paths needed for context]

### Implementation Requirements
[Specific functionality description]

### Coding Standards
- No `any` types
- Early returns instead of deep nesting
- Functions under 50 lines
- Complete error handling
- Stats/count queries use dedicated count calls, not .filter().length on paginated results

### Output Format
STATUS: SUCCESS / FAILURE
CHANGES: [list of files]
BUILD: PASS / FAIL (attach errors if FAIL)
```

### Frontend Subagent

```
## Task: Implement Phase X Frontend — [Phase Name]

### Interface Contract (frozen, do not modify)
[Complete interface contract from main agent, focus on response structures]

### Existing Files to Read
[List component file paths needed for context]

### Implementation Requirements
[Specific UI/interaction description]

### Coding Standards
- No `any` — API responses use explicit types matching the interface contract exactly
- Component props defined with interface/type
- Immutable updates (spread operator)
- Functions under 50 lines

### Output Format
STATUS: SUCCESS / FAILURE
CHANGES: [list of files]
BUILD: PASS / FAIL (attach errors if FAIL)
```

### Code Reviewer Template

```
## Task: Review Phase X

### Files to Review
[Files added or modified in this phase]

### Focus Areas
[Phase-specific checks]

### Output Format

## Code Review: Phase X

### [filename]
PASS / FAIL
- [CRITICAL/HIGH/MEDIUM/LOW] description

### Summary
APPROVED / NEEDS_FIXES

For CRITICAL/HIGH findings: fix inline with Edit tool and mark "fixed".
```

---

## Full Execution Flow

```
Main Agent starts
  │
  ├── 1. Read codebase, understand existing structure
  ├── 2. Create implementation doc (docs/feature.md)
  ├── 3. Create task list (TaskCreate)
  │
  ├── Main agent designs Phase 1 interface contract → freeze
  │
  ├── Launch in parallel:
  │     ├── Backend subagent (Phase 1)
  │     └── Frontend subagent (Phase 1)
  │
  ├── Both complete
  │
  ├── Main agent designs Phase 2 interface contract → freeze
  │
  ├── Launch in parallel:
  │     ├── Backend subagent (Phase 2)
  │     ├── Frontend subagent (Phase 2)
  │     └── code-reviewer (Phase 1)
  │
  ├── code-reviewer returns → main agent handles findings
  │
  ├── ... repeat until final phase
  │
  ├── Final phase complete → code-reviewer reviews
  ├── Update docs (all phases ✅)
  └── git commit + push
```

---

## Common Pitfalls

| Pitfall | Severity | Prevention |
|---|---|---|
| Interface contract missing fallback for missing env vars | MEDIUM | Main agent notes env dependencies and fallback requirements in contract |
| Regex special characters not escaped | HIGH | Reviewer specifically checks regex expressions |
| Stats computed from paginated results instead of dedicated count query | MEDIUM | Coding standard: always use dedicated count queries for aggregates |
| Frontend types don't match API response structure | HIGH | Main agent writes complete response JSON in contract; frontend implements directly against it |

---

## When to Use

**Use when:**
- Feature has clear frontend/backend split (API + UI)
- Touches more than 3 files
- Single phase estimated > 30 minutes

**Skip when:**
- Single-file change (just Edit directly)
- Pure backend or pure frontend (no need for parallel split)
- Urgent hotfix (fix directly, skip the process)
