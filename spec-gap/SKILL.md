---
name: spec-gap
description: Check a project's spec (SPEC.md) against its code - report every gap as a ranked table (code the spec does not cover, stale sections, missing one-line flows, numbers or steps that differ, promises the code can break, decisions the spec never mentions) and a to-do list of what the spec describes but is not built yet. Use when the user asks whether the spec still matches the code, what is left to build, before a release or hand-over, or after a lot of changes. Reports only; fixes nothing.
---

# Find spec gaps

Compare the spec with the code and report where they disagree. Change nothing: the user decides each fix (spec changes then go through `spec-update`).

Specs written with `spec-write` follow its template, but older specs may not: go by the headings in the spec itself, never by the template's section numbers.

## Scope

The working tree (uncommitted changes included) of the app's code, its config files and its prompts. Tests, evals and the frontend only if the user asks.

## Definitions

- **A module is covered** when the spec describes what it does: its own section, a row in a table, or a paragraph, even without naming the class or file.
- **A one-line flow** is a single line at the start of a module's section showing its path, each step marked with who does it:
  `triage_ticket called → load ticket (code) → pick category (LLM) → check it is allowed (code) → return`

## Look for

1. **Code not covered by the spec:** every tool, graph node, processing class or function module, the API, and anything else that makes a decision.
2. **Spec sections for things the code no longer has** (removed or renamed), and decisions-log lines that later changes contradict.
3. **Module sections without a one-line flow.**
4. **Places where the code differs from the spec:** a different order of steps, a different number (compare every threshold, limit and timeout in the spec with the config and with numbers hard-coded in the code), or a different decider (the spec says code, the code asks the LLM).
5. **Promises the code can break:** for each guarantee in the spec ("never", "always", "only when"), follow the code paths that could break it, including the LLM not doing what the prompt asks (answering in text instead of calling a tool, stopping early).
6. **Decisions in the code the spec does not mention:** a rule that rejects, routes or limits something, or a config value chosen for a reason. Skip comments that only explain how the code works.

Only report a gap after reading both sides; quote the spec words and the code line.

## Look for what is not built yet

Separately, list everything the spec describes that the code does not do yet: steps in the Steps section not done, and any feature, endpoint, rule or failure handling with no code behind it (or only part of it). These are not mistakes, they are work still to do.

If something in the spec has no code, tell which it is: **not built yet** (a later step, or never started) or **removed** (the decisions log or the code shows it was dropped, so it is a stale section above). If you cannot tell, ask.

## Report

Rank the gaps by harm: first a wrong result the spec promises cannot happen, then behaviour that differs, then missing documentation.

| Gap | Spec says | Code does | Fix in | Suggested fix |
|---|---|---|---|---|
| Unknown category goes to the general queue | §4: "a category outside the list → needs_review" | `graph/nodes.py:52` falls back to `"general"` | code | send it to `needs_review`; add a test |
| Refund limit is 1000 in code | §6: "over $500 goes to a person" | `config.yaml:12` `refund_limit: 1000` | decide | which number is right? |

**Fix in** is `spec` (the code is right, the spec is out of date), `code` (the spec is right, the code is wrong) or `decide` (the user has to choose).

Then the to-do list, in the order of the spec's Steps (or the spec's own order if it has none):

| To do | Spec | Status | What is missing |
|---|---|---|---|
| CSV export of tickets | §5 API: `GET /tickets.csv` | not started | the endpoint and its test |
| Refunds over $500 go to a person | §6 | partly built | the check exists; the hand-over to a person does not |

End with what was checked and found in order, and anything that could not be verified without running the app.
