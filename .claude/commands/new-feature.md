Start a new feature. Accepts a spec as $ARGUMENTS or prompts for one, then classifies complexity and routes to the appropriate flow.

## Step 1 — Collect the Spec

Check whether $ARGUMENTS already contains input in the structured format below (i.e. has a `### Goal`, `### Constraints`, `### What done looks like`, and `### What I'm NOT specifying` section). If it does, use it directly and skip to Step 2.

If $ARGUMENTS is empty **or** does not follow the structured format, ask the developer to provide their spec using this exact template:

```
## <Feature Title>

### Goal
<What you want to build — one short paragraph>

### Constraints
- <Constraint or requirement 1>
- <Constraint or requirement 2>
- ...

### What done looks like
- <Observable outcome 1>
- <Observable outcome 2>
- ...

### What I'm NOT specifying
- <Intentional gap 1 — things you're leaving to Claude's judgement>
- NA  ← use this if there are no intentional gaps
```

Do NOT proceed until the developer has provided input in this format. Do NOT try to infer a spec from a one-liner or rough description — always ask for the structured input first.

---

## Step 2 — Classify Complexity

Read the spec/description and classify as one of two types. **Do not ask the developer which route to take** — decide based on the signals below.

### Route A — Full Feature (spec + plan mode)

Any of these signals means Route A:
- New screen, flow, or user-facing capability that doesn't exist yet
- Requires a new module, service, or package
- Introduces a new interface/protocol boundary or repository
- Touches 3+ files across different layers (Presentation / Domain / Data)
- Involves new API endpoints or data model changes
- Has non-trivial state management
- Would take more than ~1 hour for a senior dev to implement

### Route B — Simple Task (direct execution)

All of these must be true for Route B:
- No new module, no new interface, no new screen/page
- Changes confined to 1–2 existing files or are purely additive within an existing module
- No new API endpoint, no new data model
- Obvious implementation path — no architectural decisions needed
- Would take a senior dev under ~30 minutes

**When in doubt, choose Route A.** Over-planning is cheaper than mid-implementation surprises.

---

## Route A — Full Feature

### A1 — Enrich the Spec

1. Read `CLAUDE.md` to understand project architecture invariants and conventions.

2. Build or enrich the spec with:
   - **Module classification** — which layer, which existing modules are affected
   - **Interface/Protocol boundary** — what interface the feature exposes and where it lives
   - **State design** — proposed state model(s)
   - **API contract** — endpoint(s), key request/response fields
   - **Test surface** — which scenarios must be tested, including edge cases
   - **Open questions** — anything ambiguous that needs a decision before implementation

3. Save the enriched spec to `docs/specs/<feature-name>.md`

Do NOT ask for approval on the enriched spec. Move immediately to A2.

### A2 — Architect Review

Invoke the `architect` agent with the enriched spec. The architect will:

1. **Produce design diagrams** (appended to `docs/specs/<feature-name>.md`):
   - **HLD** — component diagram showing modules, boundaries, and injected interfaces
   - **LLD** — class/interface diagram with state models and key method signatures
   - **Data Flow** — sequence diagram for primary request/response + state machine if needed

2. **Check for multiple implementation approaches**:
   - If the architecture rules fully determine a single path → architect notes this and proceeds
   - If two or more valid approaches remain after applying all rules → **STOP**. Architect presents the options with trade-offs and a recommendation and waits for the developer to choose before continuing

Do NOT enter plan mode until the architect has completed this step and any approach decision has been made.

### A3 — Enter Plan Mode

Enter plan mode with the enriched spec (including diagrams and chosen approach) as input.

The plan must cover:
- Module(s) to create or modify (with layer classification)
- Implementation steps in dependency order (interfaces/contracts first, DI/wiring last)
- Build/test verification gate at each layer
- Test strategy (which scenarios, coverage targets)
- Task breakdown (parent epic + subtasks with dependency order)

### A4 — After Plan Approval

Once the developer approves the plan:
1. Begin implementation via the approved plan
2. Track progress in `tasks/todo.md` — check off each subtask as it completes
3. Run build verification (build → tests → lint) before declaring done

---

## Route B — Simple Task

Execute directly — no spec file, no plan mode.

1. Read all relevant files before making any changes
2. Implement as described
3. Run verification after any changes:
   - Build — zero errors
   - Tests — if logic changed
   - Lint — zero violations
4. Commit with a clear message

---

## Rules That Apply to Both Routes

- Never declare done without a passing build and green tests
- Never skip tests after logic changes
- If blocked mid-implementation: stop, write the blocker clearly, do not thrash
