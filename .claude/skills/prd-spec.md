---
name: prd-spec
description: >
  Handles feature spec creation and enrichment. Triggered by /new-feature, or when the user
  says "write a spec", "create a spec for", "I want to build X". The developer provides a
  lightweight spec (Goal/Constraints/Done/Not-specifying) — Claude enriches it into a full spec
  and immediately proceeds to plan mode without waiting for spec approval. Output always goes to
  docs/specs/<feature-name>.md. Never ask for spec approval — it creates friction. Move straight
  to plan after enrichment.
---

# PRD & Spec Creation

The spec is the contract between product intent and implementation. All agents downstream
(feature-builder, test-writer, code-reviewer) use the spec's acceptance criteria as ground truth.

---

## Developer Input Format (Lightweight)

When a developer wants a new feature, they provide this lightweight spec.
Claude prompts for it if not provided. Never ask for more than this upfront.

```markdown
## [Feature Name]

### Goal
[What it does and why — 2-4 sentences]

### Constraints
- [Hard rules — architecture, layer ownership, tech choices, existing modules affected]

### What done looks like
- [Verifiable acceptance criteria — things you can check without reading code]

### What I'm NOT specifying
- [Things left to Claude's judgment]
```

**Key principle**: Developers specify the *what* and *constraints*. Claude decides the *how*.
Leave sections blank if unknown — Claude flags gaps as open questions, not blockers.

---

## Claude's Enrichment Process

After receiving the developer's lightweight spec, Claude enriches it **immediately** (no approval wait):

1. Read `CLAUDE.md` — identify which layer(s) are involved and check architecture invariants
2. Add to the spec:
   - Module classification and layer
   - Interface/protocol boundary (what interface, which module owns it)
   - Proposed state model(s)
   - API contract (endpoint, key fields, error codes)
   - Open questions (flag ambiguities — don't block, just note)
3. Save enriched spec to `docs/specs/<feature-name>.md`
4. **Immediately enter plan mode** — no spec approval step

---

## docs/ Directory Structure

```
docs/
├── PRD.md                      ← product vision, user personas, feature priorities (one file)
├── specs/
│   ├── README.md               ← overview of spec conventions
│   └── <feature-name>.md       ← one spec per feature (kebab-case filename)
└── decisions/
    ├── README.md               ← ADR index
    └── ADR-XXX-title.md        ← architecture decision records (owned by architect agent)
```

**Naming convention for feature specs**: kebab-case matching the feature or Jira epic (e.g. `user-auth.md`, `payment-flow.md`, `search-results.md`).

---

## PRD Template (`docs/PRD.md`)

Use this for the top-level product document. Written once, updated when priorities shift.

```markdown
# [Project Name] — Product Requirements Document

## Product Vision
[1 paragraph: what we are building, for whom, and why now]

## User Personas
| Persona | Description | Primary goals |
|---------|-------------|---------------|
| [Persona 1] | ... | [goal] |
| [Persona 2] | ... | [goal] |

## Feature Priority
| Priority | Feature | Why |
|----------|---------|-----|
| P0 | [Core feature] | [reason] |
| P1 | [Important feature] | [reason] |
| P2 | [Nice to have] | [reason] |

## Non-Functional Requirements
- [Performance targets]
- [Scale targets]
- [Security requirements]
- [Compliance requirements]

## Out of Scope (v1)
- [list explicitly — prevents scope creep]
```

---

## Feature Spec Template (`docs/specs/<feature-name>.md`)

Every feature gets its own spec file.

```markdown
# Feature Spec: <Feature Name>

**Status**: Draft | In Review | Approved | In Progress | Complete
**Priority**: P0 | P1 | P2
**Jira Epic**: PROJ-XXX
**Last Updated**: YYYY-MM-DD

---

## Overview

**User Story**
As a [persona], I want [goal] so that [reason].

**Acceptance Criteria** (numbered — these become test cases)
1. Given [precondition], when [action], then [expected result]
2. Given [precondition], when [action], then [expected result]
3. [Error state]: Given [network error], the user sees [error message] with [retry action]
4. [Edge case]: Given [empty state], the user sees [empty state view]

---

## User Flow

**Entry point**: [how the user/client gets to this feature]

**Happy path** (step by step):
1. [Step 1 — what triggers this, what happens]
2. [Step 2]
3. ...

**Error states**:
- Network unavailable: [handling]
- Empty response: [handling]
- [Other error]: [handling]

---

## API Contract

| Field | Value |
|-------|-------|
| Endpoint | `GET /api/v1/...` |
| Auth required | Yes / No |
| Cache | Yes (TTL: Xs) / No |

**Request parameters**:
- `param1` (required): description

**Key response fields**:
- `field1`: type, meaning
- `field2`: type, meaning

**Error codes**:
- `401`: re-auth flow
- `404`: [handling]
- `500`: [handling]

---

## State Design

**State model**:
```
enum FeatureState {
    idle
    loading
    loaded(data: Data)
    error(Error)
}
```

**Persistence**:
- In-memory only / Database (which tables/entities?) / Cache (TTL?)

---

## Module Impact

| Question | Answer |
|----------|--------|
| New module needed? | Yes — `<ModuleName>` / No |
| Layer | Domain / Data / Presentation / Core |
| Existing modules modified | [list with reason] |
| Interface changes? | Yes (⚠️ flag for architect review) / No |
| DI / composition root changes? | Yes — new registrations needed / No |

---

## Test Plan

**Use case / service test scenarios**:
1. `test_execute_whenSuccess_returnsData`
2. `test_execute_whenApiError_throwsError`
3. `test_execute_whenCacheHit_returnsWithoutApiCall`

**Manual verification steps**:
1. [Step 1]
2. [Step 2]
3. [Error scenario verification]

---

## Definition of Done
- [ ] All acceptance criteria are numbered and testable
- [ ] Architecture impact reviewed (if interface changes or new module)
- [ ] Tasks recorded in `tasks/<feature-name>.md`
- [ ] Build passes, tests pass, lint passes
```

---

## Spec → Implementation Handoff

When a spec reaches "Approved" status, provide the orchestrator with:
1. Spec file path: `docs/specs/<feature-name>.md`
2. Jira epic number (if applicable): `PROJ-XXX`
3. Acceptance criteria list (copy from spec)

The orchestrator uses the acceptance criteria as the single source of truth for all Task Specs dispatched to feature-builder and test-writer.
