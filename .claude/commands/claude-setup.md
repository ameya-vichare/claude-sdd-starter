Interactive project setup wizard. Scans the codebase, asks clarifying questions, installs tools, and generates a complete Claude Code configuration tailored to this project — CLAUDE.md, skills, agents, commands, and conventions.

Run this on any project (new or existing) to set up or refresh the Claude Code development workflow.

**Model**: Use `claude-opus-4-6` for this entire setup — it requires deep reasoning about architecture and conventions.

---

## Phase 0 — Codebase Discovery

Before asking anything, scan the project silently:

1. **File inventory**: `ls -la` at project root. Count source files by extension (exclude node_modules, .build, vendor, dist, __pycache__, .gradle, Pods, _build, deps, .mix).
2. **Platform detection** — identify language, framework, build system:

| Signal | Platform |
|--------|----------|
| *.swift, .xcodeproj/.xcworkspace, Package.swift (no AndroidManifest) | iOS / macOS (Swift) |
| *.kt, build.gradle(.kts), AndroidManifest.xml | Android (Kotlin) |
| *.kt + kotlin-multiplatform in build.gradle | Kotlin Multiplatform (KMP) |
| *.ts/*.tsx + react-native/expo in package.json | React Native / Expo |
| *.ts/*.tsx + react/next/vue/angular/svelte in package.json | Frontend (TypeScript) |
| *.js/*.jsx + react/next/vue/angular/svelte in package.json | Frontend (JavaScript) |
| *.ts + express/fastify/nestjs/hono in package.json | Backend Node.js |
| *.js + express/fastify in package.json (no *.ts) | Backend Node.js (JavaScript) |
| *.py + requirements.txt/pyproject.toml/manage.py/app.py | Python |
| *.go + go.mod | Go |
| *.rs + Cargo.toml | Rust |
| *.java + pom.xml/build.gradle (no AndroidManifest) | Java |
| *.scala + build.sbt/pom.xml | Scala |
| *.dart + pubspec.yaml | Flutter/Dart |
| *.cs + *.csproj/*.sln | C# (.NET) |
| *.rb + Gemfile | Ruby |
| *.php + composer.json | PHP |
| *.ex/*.exs + mix.exs | Elixir |
| *.c/*.cpp/*.h + CMakeLists.txt/Makefile | C/C++ |
| *.zig + build.zig | Zig |
| Multiple languages detected | Full-stack / Polyglot (ask user) |

3. **Codebase maturity**: count total source files
   - <20 → greenfield
   - 20–200 → early-stage
   - 200+ → established

4. **Existing config**: check for `.claude/`, `CLAUDE.md`, linter configs, CI configs, test directories, `.gitignore`. **Do NOT read `README.md`** — in this starter kit it describes the template, not the user's project.
5. **Git state**: is this a git repo? Existing branches?

Present findings to the user:
```
Here's what I found:
- Platform: [language] / [framework] / [build system]
- Codebase: [greenfield/early-stage/established] ([N] source files)
- Existing Claude config: [yes/no]
- Testing: [framework or "none detected"]
- Linting: [tool or "none detected"]
- CI/CD: [provider or "none detected"]
```

---

## Phase 1 — Clarifying Questions

Ask questions **one at a time** — wait for each answer before asking the next. Skip questions where discovery already answered them. Adapt wording to the detected platform.

### Always ask:
1. **"What is this project?"** — 2-3 sentence description of the product/service
2. **"Do you use a project management tool?"** — Linear, Jira, or none
3. **"What's your build, test, and lint setup?"** — If detected, confirm: "I detected [X] — correct?"

### For existing codebases (>20 files):
4. **"What's your primary goal?"**
   - (a) Improve code quality and engineering practices over time
   - (b) Ship features fast
   - (c) Both — quality on critical paths, speed elsewhere
5. **"Are you rebuilding from a legacy codebase?"** — If yes, where is the old code? (enables old-app-researcher agent)

### For greenfield (<20 files):
4. **"What's your approach?"**
   - (a) Scalable and modular from day one — clean architecture, clear boundaries
   - (b) Start simple, refactor as patterns emerge
   - (c) In between
5. **"Solo developer or team?"**

### Platform-specific (ask only if not obvious):
- **Mobile (iOS)**: SwiftUI or UIKit? Min iOS version? State management approach?
- **Mobile (Android)**: Compose or XML? Min API level? Architecture Components or custom?
- **Kotlin Multiplatform**: Which targets? (iOS, Android, Web, Desktop) Shared UI (Compose Multiplatform) or native UI per platform?
- **React Native / Expo**: Expo managed or bare workflow? Navigation library? (React Navigation / Expo Router) State management?
- **Frontend**: State management? (Redux, Zustand, Pinia, signals, etc.) SSR or SPA? Styling approach? (Tailwind, CSS Modules, etc.)
- **Backend (Node/Python/Java/Scala)**: Database? ORM? Auth approach? REST or GraphQL?
- **PHP**: Framework? (Laravel, Symfony, Slim, vanilla) Laravel version? API or full-stack (Blade/Livewire/Inertia)?
- **Elixir**: Phoenix or plain OTP? Live View or API-only? Database? (Postgres/Ecto assumed — confirm)
- **Go**: Standard `net/http` or framework? (Gin, Echo, Fiber, Chi) DI approach? (wire, fx, manual)
- **Rust**: Async runtime? (Tokio, async-std) Web framework? (Axum, Actix, Rocket) ORM? (Diesel, SQLx, SeaORM)
- **C/C++**: Build system? (CMake, Bazel, Make) Testing framework? (GTest, Catch2, Doctest) Platform target?
- **Scala**: Framework? (Play, http4s, ZIO HTTP) Effect system? (Cats Effect, ZIO) Build tool? (sbt, Mill)
- **Zig**: Application type? (CLI, embedded, library) Build system config?
- **Flutter**: State management? (Riverpod, Bloc, Provider)
- **Full-stack**: Monorepo or separate repos? Shared types?

### Architecture:
6. **"Any specific patterns to enforce?"** (MVVM, Clean Architecture, Hexagonal, etc.)
   - If blank, recommend based on platform:
     - iOS/Android/Flutter → MVVM + Clean Architecture (Domain/Data/Presentation)
     - Kotlin Multiplatform → shared domain + data layer; platform-native presentation per target
     - React Native / Expo → feature-based modules, React Navigation, state management layer
     - Frontend → feature-based modules with state management layer
     - Backend (Node/Python/Java) → layered (Controller → Service → Repository) or hexagonal
     - PHP (Laravel) → MVC with Service layer; Repositories for complex queries
     - Elixir (Phoenix) → context-based (Phoenix Contexts as bounded domains), Ecto schemas
     - Go → package-per-feature with explicit interfaces; avoid global state
     - Rust → module-per-domain, trait-based abstractions, error types per domain
     - Scala → functional layers (HTTP → Service → Repository) with effect types
     - C/C++ → header/source separation, PIMPL for encapsulation, no global mutable state
     - Zig → comptime-driven generics, explicit allocators, no hidden control flow

After all questions are answered, summarize understanding and confirm before proceeding.

---

## Phase 2 — Tool Installation

### 2a — Context7 (always recommend)
```
I'd like to install Context7 — it gives me real-time documentation for your libraries
and frameworks so I write more accurate, up-to-date code.

Please run:  npx ctx7 setup
```
Wait for confirmation. After install, **use Context7 throughout the remaining phases** to look up current best practices for the user's framework and language version.

### 2b — Platform-specific MCP tools
Based on detected platform, suggest relevant tools in a batch:

| Platform | Tool | Purpose | When to suggest |
|----------|------|---------|----------------|
| iOS | XcodeBuildMCP | Build, test, screenshot | Always for iOS |
| All | Linear MCP | Ticket management | If user has Linear |
| All | Figma MCP | Design reference | If user mentions design |
| Frontend | browser-tools-mcp | Browser testing | If UI project |

For each tool the user agrees to, provide the install command and wait for confirmation.

### 2c — Git init
If no git repo detected, ask: "Can I initialize a git repo here?"

---

## Phase 3 — Write PRD.md

Using the user's project description, write `docs/PRD.md`:

```markdown
# [Project Name] — Product Requirements Document

**Status**: Draft
**Last Updated**: [today's date]

## Product Vision
[1 paragraph — what, for whom, why. Based on user's description]

## User Personas
| Persona | Description | Primary goals |
|---------|-------------|---------------|
| ... | ... | ... |

## Feature Priority
| Priority | Feature | Why |
|----------|---------|-----|
| P0 | [core] | ... |
| P1 | [important] | ... |

## Non-Functional Requirements
- [Performance: platform-appropriate targets]
- [Security: platform-appropriate requirements]
- [Accessibility: if applicable]

## Out of Scope (v1)
- [explicit list]
```

Ask user any clarifying questions needed to fill this in properly. Present draft and wait for approval before continuing.

---

## Phase 4 — Generate CLAUDE.md

**Target: ~200 lines, project-specific, zero placeholder text.**

Replace the existing generic CLAUDE.md with a project-specific version containing these sections:

### Section 1 — Feature Development Workflow (~30 lines)
Same Route A/B flow diagram from the starter. Adapt:
- Replace "Jira" with user's PM tool (or remove PM references if none)
- Keep Route A/B classification signals

### Section 2 — Workflow Orchestration (~30 lines)
Keep the 6 principles as-is — they're platform-agnostic:
1. Plan Mode Default
2. Subagent Strategy
3. Self-Improvement Loop
4. Verification Before Done
5. Demand Elegance (Balanced)
6. Autonomous Bug Fixing

### Section 3 — Architecture Invariants (~40 lines)
**CRITICAL**: Fill in REAL rules based on user's answers. Examples by platform:

**Mobile (iOS/Android/Flutter)**:
- Entry point and navigation ownership
- ViewModel pattern specifics (protocol-based? observable?)
- Layer rules: Presentation → Domain → Data
- DI approach and where it's wired
- State management rules (state enums, no scattered booleans)
- Async pattern (async/await, coroutines, Futures)
- Persistence approach and module boundary rules

**Kotlin Multiplatform**:
- Shared module owns: domain models, use cases, repositories (interfaces), business logic
- Platform modules own: UI, platform-specific implementations, DI wiring
- `expect`/`actual` for platform-specific behaviour — minimise its surface area
- No UI framework imports in shared module

**React Native / Expo**:
- Navigation ownership (navigator vs screen component)
- State management (global store vs local component state — when to use each)
- Native module usage rules (prefer Expo modules, custom native only when unavoidable)
- Platform-specific code (`Platform.OS` guards vs `.ios.ts`/`.android.ts` splitting)
- No business logic in screen components

**Frontend (React/Vue/Angular/Svelte)**:
- Component hierarchy and routing
- State management rules (global vs local, when to use each)
- API layer rules (centralized client, error handling)
- Styling approach and rules; no business logic in components
- Module/feature folder structure

**Backend (Node/Python/Java/Scala)**:
- Layer rules: Controller/Handler → Service → Repository
- DI/IoC approach; no service locator
- Database access rules (ORM, raw SQL, repository pattern)
- Error handling and propagation rules
- Middleware/interceptor conventions; auth/authz rules

**PHP (Laravel)**:
- Controller → Service → Repository (no Eloquent queries outside repository layer)
- Dependency injection via service container; no `app()` helper inside business logic
- Eloquent models are data structures only — no business logic on models
- Form Requests for all input validation; never validate in controller
- Queue jobs for anything >200ms; no synchronous side effects in request lifecycle

**Elixir (Phoenix)**:
- Phoenix Context = bounded domain; no cross-context direct function calls (use pub/sub or explicit interfaces)
- Ecto schemas represent DB shape only — changesets validate, contexts transform
- No business logic in controllers or LiveView callbacks — delegate to contexts
- Pattern match on `{:ok, result}` / `{:error, reason}` everywhere; no exceptions for control flow

**Go**:
- Explicit interfaces at package boundaries; no global state
- Error returned, never panicked (except program startup)
- Context propagation for cancellation and deadlines — first arg of every public function
- No `init()` functions with side effects

**Rust**:
- Explicit error types per domain (no `Box<dyn Error>` in library code)
- Trait-based abstractions; no concrete type leakage across module boundaries
- No `unwrap()`/`expect()` in production paths — propagate with `?`
- Async runtime chosen once; no mixing Tokio + async-std

**Scala**:
- Effect type chosen once (Cats Effect IO or ZIO) — never mix
- Repository returns `F[A]` not concrete types; inject via type class or trait
- No `throw` — use `EitherT`/`IO.raiseError`/`ZIO.fail`

**C/C++**:
- Headers declare interfaces; source files implement — no implementation in headers (except templates)
- RAII everywhere — no manual `delete`, use smart pointers
- No global mutable state; no `extern` globals in library code
- Error via return codes or exceptions — pick one per module boundary, never mix

**Zig**:
- Explicit allocator passed to every function that allocates — no implicit allocation
- Errors returned via error union (`!T`) — never ignored
- No hidden control flow; no implicit coercions

**Use Context7** to verify conventions match current framework best practices.

### Section 4 — Build & Verification (~15 lines)
Project-specific commands:
```
Build: [exact command]
Lint: [exact command]
Test: [exact command]
Run: [exact command]
```
Standard verification sequence. When build fails protocol.

### Section 5 — Git Conventions (~15 lines)
What to commit/ignore — adapted for their stack's artifacts.

### Section 6 — Task Management (~10 lines)
Same pattern as starter.

### Section 7 — Model Selection + Thinking Modes (~15 lines)
Same tables, adapt "when to use" examples to their domain.

### Section 8 — @ Reference Quick Access (~15 lines)
Updated paths for ALL generated files.

**Rules**:
- Every invariant must be specific and enforceable
- Use the user's actual framework/tool names
- No placeholder text — if something is unknown, omit the section entirely
- Reference skills by name (e.g., "see `swift-conventions` skill" not inline rules)

---

## Phase 5 — Generate Skills

Create each file in `.claude/skills/` with YAML frontmatter (name, description, trigger conditions).

**Use Context7** for each skill to look up current best practices, API patterns, and idioms for the user's language/framework version. This makes the skills rich and accurate.

### 5a — architecture-guard.md
Project-specific architecture rules. Sections:
- **Layer hierarchy** — specific to their pattern (draw the dependency diagram)
- **Module boundary rules** — specific to their module system
- **DI rules** — specific to their DI container/approach
- **State management rules** — specific to their state solution
- **Networking/API rules** — HTTP client, error handling, retry patterns
- **Security rules** — token storage, input validation, PII handling (platform-appropriate)
- **Pre-structural-change checklist** — 8-12 items to verify before any structural change

### 5b — debugger.md
Platform-specific debugging guide. Sections:
- **Diagnosis protocol**: reproduce → capture logs → classify (crash/logic/UI/network) → identify layer → root cause → fix → verify
- **Log capture**: platform-specific (Xcode console / logcat / browser devtools / server logs / structured logging)
- **Crash/error classification**: specific to their language runtime (null pointer, type error, unhandled promise, panic, etc.)
- **Common failure patterns**: 8-10 platform-specific patterns with diagnosis steps (table format: symptom → likely cause → fix)
- **Tools to use**: platform-specific debugging tools
- **Never do**: platform-specific anti-patterns (e.g., never `console.log` and leave it, never disable SSL to debug, never `catch {}` to silence errors)

### 5c — [language]-conventions.md
Language and framework coding standards. **Use Context7 heavily here.** Sections:
- **Naming conventions**: types, functions, variables, files, modules — language-idiomatic
- **File structure**: how to organize files within a module/feature
- **Pattern templates**: concrete code patterns for their architecture (ViewModel, Repository, UseCase, Component, Service, etc.) — with actual code shapes in their language
- **Async/concurrency idioms**: language-specific (async/await, coroutines, Promises, channels, etc.)
- **Error handling patterns**: language-specific
- **Access control / visibility**: language-specific (public/private/internal, exports, etc.)
- **Forbidden patterns**: 10-15 specific anti-patterns for their language/framework with what to use instead

### 5d — test-writing.md
Testing conventions. Sections:
- **What must be tested** (by layer): domain 100%, data 80%+, presentation varies
- **What does NOT need testing**: framework internals, trivial getters, UI layout
- **Test naming**: convention for their test framework (`test_method_context_expected` or `describe/it` or `@Test fun`)
- **Test structure**: Given/When/Then with their framework's actual syntax
- **Mock/stub approach**: specific to their mocking library (Mockito, Jest mocks, unittest.mock, gomock, etc.)
- **Fixture/factory patterns**: how to create test data in their language
- **Coverage requirements**: domain/data ≥80%, no file drops >2%
- **Test file organization**: where test files live, naming convention

### 5e — [tool]-verification.md
Build verification specific to their toolchain. Sections:
- **Build**: exact commands, common errors and fixes
- **Test**: exact commands, how to run specific tests/modules
- **Lint**: exact commands, how to auto-fix
- **Run locally**: exact commands (simulator, emulator, dev server, etc.)
- **Screenshot/UI verification**: if applicable (simulator screenshot, browser screenshot, etc.)
- **Coverage reporting**: how to generate and read coverage
- **Definition of "build verified"**: checklist of what must pass

### 5f — module-creation.md (only if project uses explicit modules)
Step-by-step module creation guide:
- Module type classification
- Scaffold steps (exact commands for their package system)
- Required files in order (interfaces first)
- DI wiring steps
- Test setup
- Common mistakes

Only generate if: Swift packages, Gradle modules (Android/KMP), npm workspaces, Go packages with clear module boundaries, PHP Composer packages, Elixir umbrella apps (mix umbrella), Scala sbt multi-project, C/C++ CMake subprojects, or user explicitly requested modular architecture.

### 5g — prd-spec.md (customize existing)
Read the existing `prd-spec.md` and adapt:
- Replace generic examples with platform-specific state patterns
- Replace "Jira" with user's PM tool
- Keep the lightweight spec format and enrichment process

### 5h — pr-checklist.md (customize existing)
Read the existing `pr-checklist.md` and adapt:
- Add platform-specific checks (accessibility for mobile, hydration for SSR, migration safety for DB changes)
- Reference the correct conventions skill by name
- Reference the correct verification commands

---

## Phase 6 — Generate Agents

Create/customize files in `.claude/agents/`. Each agent has YAML frontmatter with `model:` field.

### Core agents (customize existing 5):

#### orchestrator.md (model: claude-sonnet-4-6)
Read existing and adapt:
- **PM tool integration** (critical if user has Linear/Jira):
  - Configure the project/team ID for the PM tool
  - Epic creation: title, description (plan summary + acceptance criteria), priority, state (Backlog)
  - Subtask creation: linked to epic via `parentId`, with priority, layer, files, acceptance criteria
  - Dependency setting: `blockedBy` relationships between subtasks matching the implementation order
  - Status transitions: Backlog → In Progress → Done/Blocked/Cancelled with comments at each milestone
  - Assignee mapping if team members are known
  - Record all issue IDs in task files
  - If no PM tool → remove PM sections entirely
- Reference their verification skill for build commands
- Keep epic decomposition rules, task spec format, parallelism rules
- Adapt worktree/branch commands for their git workflow

#### architect.md (model: claude-opus-4-6)
Read existing and adapt:
- Layer hierarchy specific to their architecture
- Module boundary decision tree for their module system
- DI evaluation rules for their DI approach
- Diagram examples adapted to their domain (keep Mermaid format)

#### feature-builder.md (model: claude-sonnet-4-6)
Read existing and adapt:
- Reference their `[language]-conventions` skill
- Reference their `[tool]-verification` skill
- Platform-specific implementation order:
  - Mobile: interfaces → models → repository → viewmodel → view → DI
  - Frontend: types → API layer → state/store → components → routing
  - Backend: interfaces → models → repository → service → controller → DI
- Platform-specific forbidden patterns

#### code-reviewer.md (model: claude-sonnet-4-6)
Read existing and adapt:
- Reference their conventions and architecture-guard skills
- Add platform-specific review dimensions (e.g., accessibility, performance, security)
- Keep severity format (Blocker/Major/Minor/Nit)

#### test-writer.md (model: claude-sonnet-4-6)
Read existing and adapt:
- Reference their `test-writing` skill
- Use their test framework syntax in examples
- Use their mock/stub library

### New agents to create:

#### ci-fixer.md (model: claude-sonnet-4-6)
```yaml
---
model: claude-sonnet-4-6
---
```
Sections:
- **Identity**: Diagnoses and fixes CI/build failures autonomously. Never guesses from partial errors.
- **Trigger**: Build failure, test failure, lint violation increase, coverage drop >2%
- **Diagnostic approach**: Read FULL error output → classify failure type → reproduce locally → fix root cause (not symptom)
- **Common fixes**: 8-10 platform-specific patterns (missing dependency, type error, test failure, lint violation, migration error, etc.) with diagnosis and fix for each
- **Lessons protocol**: After every fix, add entry to `tasks/lessons.md` (failure, root cause, fix, prevention rule)
- **Escalation**: After 3 fix attempts → write `CI_FIX_BLOCKED` with what was tried and what's needed
- **Commit format**: `[ci-fix] [<module>] <description>`
- **Never**: Guess from partial errors, disable linters/checks to "fix" failures, introduce new warnings

#### refactor-agent.md (model: claude-sonnet-4-6)
```yaml
---
model: claude-sonnet-4-6
---
```
Sections:
- **Identity**: Improves code quality without changing behavior. Runs after code-reviewer approval on non-trivial features (>~150 lines changed).
- **Pre-refactor**: Confirm all tests green, note coverage %, read conventions skill, read lessons
- **Scope IN**: Reduce cognitive complexity, extract duplication, simplify conditionals, improve names, consolidate state, use language-idiomatic patterns, extract magic numbers, parallelize independent async calls
- **Scope OUT**: Change public API, change observable behavior, add features, fix bugs, modify tests, change DI wiring, architectural restructuring
- **Elegance test**: "Knowing everything I know now, is this what I would write from scratch?"
- **Verification**: All tests green, lint clean, coverage not dropped, diff review confirms no behavior change

#### [language]-researcher.md (model: claude-sonnet-4-6)
```yaml
---
model: claude-sonnet-4-6
---
```
Sections:
- **Identity**: Research agent for [platform] topics. Never writes production code. Returns structured summaries.
- **Research protocol**: Clarify exact question → search docs (use Context7) → cross-reference against project constraints → return structured output
- **Output format**: API/Pattern name | Available Since | Compatible with [min version] | Summary (2-3 sentences) | Key Caveats | Recommended Approach | Code Shape (concept only) | Sources
- **Project constraints**: List specific constraints (min OS/language version, no specific libraries without review, framework preferences)
- **Never**: Write production code, recommend patterns that violate architecture-guard, recommend deprecated APIs

#### old-app-researcher.md (model: claude-sonnet-4-6) — ONLY if user said they're rebuilding
```yaml
---
model: claude-sonnet-4-6
---
```
Sections:
- **Identity**: Reads legacy codebase at `[configured path]`. Never writes code. Extracts behavioral contracts, data shapes, edge cases, anti-patterns.
- **Path check**: First verify legacy repo exists at configured path. If not found, return unavailability message.
- **Signal classification**: Classify every finding as:
  - `CARRY FORWARD` — proven behavior to replicate
  - `ANTI-PATTERN` — explicitly NOT to repeat (explain why)
  - `CONTEXT ONLY` — background information
- **Output format**: Question answered | Files read | Findings (with classification) | Anti-patterns (name, what, why, what instead) | Recommendations
- **Never**: Copy code verbatim, recommend anti-patterns found, read outside configured path, make up findings

---

## Phase 7 — Generate Commands

### 7a — new-feature.md (customize existing)
Read existing and adapt:
- Enrichment step references the correct skills (architecture-guard, [language]-conventions)
- Build verification uses the correct commands from [tool]-verification
- **PM tool integration in A4 (after plan approval)**:
  - If PM tool configured: create epic → create subtasks with parentId → set blockedBy dependencies → set priority → update status at each milestone
  - If no PM tool: remove PM references, track only in tasks/ files
- Route A/B signals adapted if platform warrants it

### 7b — [pm-tool]-do.md (only if user has Linear/Jira)

Template for Linear:
```markdown
Execute an existing Linear ticket. Accepts a ticket ID as $ARGUMENTS.

## Step 1 — Fetch the Ticket
Use the Linear MCP to fetch the ticket by ID: $ARGUMENTS
Read the title, description, acceptance criteria, labels, and any subtasks.
Mark the ticket as "In Progress".

If $ARGUMENTS is empty, ask: "Which ticket? Provide the Linear ticket ID (e.g., PROJ-123)."

## Step 2 — Classify
Read the ticket and classify:
- **Feature ticket** (acceptance criteria, new functionality) → Route A
- **Bug ticket** (broken behavior, reproduction steps) → diagnose and fix directly
- **Simple task** (chore, config, small refactor) → Route B

## Step 3 — Execute

### Feature ticket → Route A:
1. Check `docs/specs/` for existing spec matching this ticket
   - If found: read and proceed to architect review
   - If not: build spec from ticket description using prd-spec skill, save to `docs/specs/`
2. Follow Route A flow from /new-feature (architect → plan → implement)
3. On plan approval: create subtasks in Linear linked to this ticket as parent (`parentId`), with:
   - One subtask per implementation step
   - `blockedBy` dependencies matching implementation order
   - Priority inherited from parent ticket
   - Assignee if known
4. Update subtask statuses at each layer milestone (In Progress → Done)
5. Mark parent ticket `In Review` when PR is opened

### Bug ticket:
1. Read bug description and reproduction steps
2. Reproduce the bug — verify it exists
3. Diagnose root cause (read logs, trace the code path)
4. Fix and verify (build + test + manual verification)
5. Update ticket: root cause, fix description, verification steps
6. Mark ticket Done

### Simple task → Route B:
1. Implement directly
2. Verify (build + test + lint)
3. Commit with ticket ID in message
4. Mark ticket Done
```

For Jira: same structure but use Jira MCP or `gh` commands as appropriate. Name the file `jira-do.md`.

### 7c — Utility commands (generate all that apply):

**build.md**: "Build the project. Run: `[exact build command from Phase 1]`. Report result."

**test.md**: "Run tests. Accepts optional $ARGUMENTS for test filtering. Run: `[exact test command]`. If $ARGUMENTS provided, filter to that module/file."

**fix-build.md**: "Diagnose and fix the current build failure. Invoke the ci-fixer agent with the current error output."

**lint.md**: "Run the linter. Run: `[exact lint command]`. If violations found, fix them automatically where possible."

---

## Phase 8 — README & Verification

### 8a — Generate README.md
Write a project README that includes:
- **What this project is** (from PRD, 2-3 sentences)
- **Getting started** — prerequisites, setup steps, how to build/run
- **Claude Code setup** — explain the agentic workflow:
  - `/new-feature` — start any new feature
  - `/[pm-tool]-do` — work on a ticket (if applicable)
  - `/build`, `/test`, `/lint` — utility commands
  - How agents collaborate (brief: orchestrator → architect → feature-builder → test-writer → code-reviewer)
- **Project structure** — directory layout with descriptions
- **Architecture** — brief description pointing to CLAUDE.md for details

### 8b — Verify all generated files
Review every file and confirm:
- [ ] CLAUDE.md is ~200 lines, concise, zero placeholder text
- [ ] All skills reference the correct language, framework, and tools
- [ ] All agents reference the correct skills by filename
- [ ] All commands use the correct build/test/lint commands
- [ ] docs/PRD.md is filled in with real content
- [ ] README accurately describes the setup
- [ ] All `@ Reference Quick Access` paths in CLAUDE.md are correct
- [ ] Cross-references between files are consistent (no broken references)
- [ ] `.gitignore` covers the project's artifacts

### 8c — Present summary
```
Setup complete! Here's what was generated:

CLAUDE.md             — [N] lines, project-specific conventions
docs/PRD.md           — product requirements

Skills:
  - architecture-guard.md
  - debugger.md
  - [language]-conventions.md
  - test-writing.md
  - [tool]-verification.md
  - module-creation.md (if applicable)
  - prd-spec.md (customized)
  - pr-checklist.md (customized)

Agents:
  - orchestrator.md (sonnet) — coordinates all agents
  - architect.md (opus) — evaluates structure, writes ADRs
  - feature-builder.md (sonnet) — writes production code
  - code-reviewer.md (sonnet) — reviews PRs
  - test-writer.md (sonnet) — writes tests
  - ci-fixer.md (sonnet) — fixes build failures
  - refactor-agent.md (sonnet) — improves code quality
  - [language]-researcher.md (sonnet) — platform research
  - old-app-researcher.md (sonnet) — legacy codebase research (if applicable)

Commands:
  /new-feature         — start any new feature
  /[pm-tool]-do        — work on a ticket (if applicable)
  /build, /test, /lint — utility commands
  /fix-build           — diagnose build failures

To start building:  /new-feature
```

---

## Rules for the Entire Setup

1. **Never generate placeholder text** — if you don't know something, ask or omit
2. **Use Context7** for every skill that references language/framework specifics
3. **Ask before overwriting** — if a file already exists with custom content, show the diff and confirm
4. **One phase at a time** — complete each phase before moving to the next
5. **Verify at the end** — don't skip Phase 8b
6. **Adapt the questions** — these are guidelines, not a rigid script. If context makes a question irrelevant, skip it. If something important is missing, ask it.
7. **Keep CLAUDE.md under ~200 lines** — push details into skills, not into CLAUDE.md
8. **Every file must be self-contained** — an agent reading one file should not need to read another to understand its own role (though it may reference other files for project context)
