## Feature Development Workflow

This is the canonical flow for building any new feature. Every developer on the team follows this.

```
1. /new-feature [rough description]
   └── Claude classifies complexity automatically (no developer input needed)
   │
   ├── Route A — Full Feature (new screen / module / service boundary)
   │   └── Claude enriches spec → saves to docs/specs/<feature>.md
   │   └── Claude enters plan mode immediately (no spec approval step)
   │   └── Developer approves plan
   │   └── Orchestrator records task breakdown in tasks/<feature>.md
   │   └── Implementation: verify build/tests after every layer
   │   └── test-writer → code-reviewer → (optional) refactor-agent → PR
   │
   └── Route B — Simple Task (≤2 files, no new module/service, <30 min)
       └── Claude implements directly — no spec file, no plan mode
       └── Run tests + lint before done
       └── Commit
```

**Route A signals:** new screen, new module, new service boundary, 3+ files across layers, new API endpoint, complex state management, new data model
**Route B signals:** additive change to existing file(s), no new module/service, obvious path, no architectural decisions needed
**When in doubt → Route A**

**Entry point for any new feature: `/new-feature`**
**Entry point for an existing Jira ticket: open it, copy the description into `/new-feature`**

---

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Maintain only the **top 10 most important lessons**; prune lower-impact or resolved entries to prevent token bloat
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behaviour between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "Is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding. Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

---

## Architecture Invariants

> Fill in your project's non-negotiable architectural rules here.
> Examples below — replace with what applies to your stack.

- Separate concerns: presentation / domain / data layers
- Protocol/interface in feature module, implementation in separate module — always
- DI wired at app/service level only — no service locators inside features
- No business logic in presentation layer
- All async code uses the project's standard async pattern (async/await, Promises, etc.)
- Unit test every domain service and data layer — no exceptions

---

## Git — What to Commit and What to Ignore

### Always commit
- Source files, config, package manifests
- Shared CI/CD config files

### Never commit
- Generated build artifacts
- IDE per-user state files (`.idea/`, `.vscode/settings.json` if user-specific)
- Dependency caches (`.gradle/`, `node_modules/`, `.venv/` etc.)
- `.DS_Store` — macOS metadata
- `*.key`, `*.pem`, `*.p12` — credentials (use CI secrets)
- `.env`, `.env.*` — environment variables with secrets
- **Any media file > 1 MB** unless explicitly needed

### When staging files
Before `git add`, run `git status` and check:
- No secrets or credentials
- No media files > 1 MB
- No IDE user-state directories

---

## Task Management
1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plans**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles
- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---

## Build Verification (Required Before Declaring Done)

**NEVER declare a task complete, mark a checklist item done, or hand off to the next agent without running a build/test verification. This is non-negotiable.**

### Standard Verification Sequence
1. Build / compile — zero errors
2. Lint — zero violations
3. Tests — all pass after any logic change

### When Build Fails
1. Read the full error output — never guess from a partial line
2. Diagnose root cause and fix
3. Do NOT hand off or mark done with a failing build

---

## Model Selection

| Model | When to use | Agents |
|-------|-------------|--------|
| `claude-opus-4-6` | Complex planning, architecture decisions, deep research | orchestrator, architect |
| `claude-sonnet-4-6` | Implementation, reviews, test writing, refactors, CI fixes | feature-builder, code-reviewer, test-writer, refactor-agent |

## Thinking Modes

| Keyword | When to use | Don't use for |
|---------|-------------|---------------|
| `ultrathink` | New module design, cross-feature architecture, protocol/interface changes affecting 3+ modules | Single-file changes |
| `think hard` | Single service state design, async correctness, DI for one module, subtle bug | Mechanical tasks |
| *(none)* | Add a test case, fix a lint error, rename a file, scaffold from a clear template | — |

---

## @ Reference Quick Access

- `@tasks/lessons.md` — **read at every session start** (top 10 lessons)
- `@tasks/todo.md` — current work in progress
- `@docs/PRD.md` — product requirements
- `@docs/specs/<feature>.md` — feature spec for current feature
- `@.claude/skills/prd-spec.md` — spec writing guide
- `@.claude/skills/pr-checklist.md` — pre-PR verification checklist
- `@.claude/agents/orchestrator.md` — orchestrator agent instructions
- `@.claude/agents/architect.md` — architect agent instructions
