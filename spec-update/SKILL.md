---
name: spec-update
description: Update an existing spec (SPEC.md) when the design changes - a request that is not in the spec, a new feature, the next step about to be built, or a decision changed during the build. Flags out-of-spec requests before any code, edits the right section and adds a decisions-log line, in the same change as the code.
---

# Update the spec

The spec must always say what the code does. Every design change goes into the spec in the same change as the code, never "later". The template is in the `spec-write` skill.

## When

1. **A request is not in the spec, or goes against it.** Flag it before writing any code, then stop and wait:
   > "CSV export is not in the spec. Should I add it to §3.7 API first?"
   > "This goes against §3.5.1: a name match is never a HIT. Should I change that rule?"

   If the user says no, do not build it.
2. **A step is about to be built.** Fill in its module section now (the steps, who does each, why), not earlier.
3. **A new feature.** Add its module section and its steps, surface first, like `spec-write`.
4. **The design changed during the build** (a test showed the plan was wrong, a better way turned up). Update the spec with the code that changes it.

## How

- **Change only what changed.** Edit the section it belongs in; do not rewrite or reorder the rest.
- **Follow the spec's own layout.** If an older spec does not match the template, fit the change into its sections instead of restructuring it.
- **Keep the one-line flow true.** If a module's steps change, its flow line changes too.
- **Remove what is no longer true.** The spec describes the design as it is now; the history goes in the decisions log.
- **Add a decisions-log line** for every design decision, in the log's existing date format:
  `| 11-18 | LLM failure in screening → REVIEW, keeping what was found | Before, it crashed the run (500) and lost the totals. |`
  The why says what forced it: what was wrong before, or which test or case showed it.
- **Update the Steps section:** mark the step done, and detail the next one only when it is next.
- **Show the spec change with the code change**, so the user reviews both together.
