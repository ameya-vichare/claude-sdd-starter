# Lessons Learned

Top 10 lessons from corrections and experience. Keep this pruned — quality over quantity.

> Claude reads this at every session start. Update after any correction from the user.

---

<!-- Add lessons here as you learn them. Format:
## [N]. Short title
**Rule**: What to do (or not do).
**Why**: The reason this matters.
**How to apply**: When this kicks in.
-->

## 1. Verify interface dependencies are merged before implementation starts
**Rule**: Before feature-builder begins, confirm all interface/contract changes from dependent tasks are merged into the working branch.
**Why**: Starting implementation on unstable interfaces causes rework and merge conflicts mid-feature.
**How to apply**: In every Task Spec, list inputs under "Inputs: (confirm merged)" and check before spawning the agent.

## 2. Route A issues require plan approval before any implementation
**Rule**: A structured spec in a ticket = Route A signal. Mandatory flow: enrich spec → architect review → plan mode → wait for developer approval → implement.
**Why**: Skipping approval means no checkpoint to catch wrong assumptions before code is written.
**How to apply**: In `/new-feature`, after A3 completes, always STOP and wait. Never start A4/A5 without explicit approval.
