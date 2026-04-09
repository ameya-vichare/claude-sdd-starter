---
name: pr-checklist
description: >
  Defines what Claude must verify before marking any task as complete or ready for PR.
  Use this skill before declaring a task done, before asking for a PR review, after
  implementing a feature or fix, or when the user says "is this ready?", "check this",
  "review this before PR", or "verify this is complete". This checklist is mandatory —
  a task is NOT done until every applicable item passes.
---

# PR Checklist

A task is **not complete** until every applicable section below passes.
Work through each section in order. If any item fails, fix it before proceeding.
Never mark a task as done and leave failing items for the reviewer to catch.

---

## 0 — Self-Assessment (Run First)

Before checking anything else, ask:

> "Would a staff engineer at a company that ships to millions of users approve this PR
> without requesting changes?"

If the honest answer is no — fix the issues first, then run this checklist.

---

## 1 — Architecture Compliance

Read `CLAUDE.md` invariants and verify:

```
[ ] Layer boundaries are respected — no outer layer importing inner layer directly
[ ] All new dependencies injected via interfaces — no concrete types in constructors
[ ] DI/composition root is the only place where concrete types are instantiated
[ ] No circular dependencies introduced
[ ] Module structure follows established conventions
[ ] No service locator pattern (Container.resolve() / static singletons inside modules)
```

**If any box is unchecked → do not submit PR. Fix the violation.**

---

## 2 — Correctness

```
[ ] Implementation matches all acceptance criteria in the feature spec
[ ] All error paths from the spec are handled
[ ] No silent failures — errors are propagated or logged appropriately
[ ] No regression in existing functionality
```

---

## 3 — Tests

```
[ ] Every new public method has at least one test
[ ] Tests cover: happy path, error path, edge cases from spec
[ ] All tests follow Given/When/Then format
[ ] All tests use mock/stub types — no real network/DB/filesystem calls
[ ] Test names follow: test_<method>_<context>_<expectedOutcome>()
[ ] Coverage for this module did not drop
[ ] All tests pass locally before PR is opened
```

---

## 4 — Code Quality

```
[ ] No dead code committed
[ ] No commented-out code blocks
[ ] No TODO/FIXME without a linked ticket
[ ] No hardcoded secrets, credentials, or API keys
[ ] Variable and function names are clear and intention-revealing
[ ] Complex logic has a brief comment explaining *why*, not *what*
```

---

## 5 — Security & Privacy

```
[ ] No PII in logs or analytics payloads
[ ] No sensitive data (tokens, passwords, card info) in logs
[ ] Credentials stored in a secret manager or environment variables — not in source code
[ ] Input validation at system boundaries (user input, external API responses)
[ ] No new API keys or secrets hardcoded in source files
```

---

## 6 — Performance (for critical paths)

```
[ ] No synchronous blocking calls on the main/event thread
[ ] No N+1 query patterns introduced
[ ] Heavy operations are async or offloaded appropriately
[ ] Parallel operations use concurrent primitives, not sequential awaits
```

---

## 7 — PR Description Quality

Before opening the PR, the description must include:

```
[ ] What changed (1-2 sentence summary)
[ ] Why it changed (the problem being solved)
[ ] How to test it manually (step-by-step for reviewer)
[ ] Link to the corresponding Jira ticket
[ ] List of files intentionally NOT changed (if a reviewer might wonder why)
```

---

## 8 — Final Gate

```
[ ] Lint passes with zero errors
[ ] Build passes with zero errors
[ ] All tests pass
[ ] No new compiler/interpreter warnings introduced
[ ] CI checks green (or failure understood and tracked)
```

---

## Signing Off

Once all applicable sections pass, you may declare the task complete.

Include this sign-off in your task summary:
```
PR Checklist: PASSED
Architecture: ✅  Correctness: ✅  Tests: ✅  Security: ✅
```

If any section was skipped with justification, note it explicitly:
```
PR Checklist: PASSED (with exceptions)
Performance: DEFERRED — tracked in TEAM-XXX
```
