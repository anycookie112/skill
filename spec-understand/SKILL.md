---
name: spec-understand
description: Read and explain a project's spec (SPEC.md) top-down, answer "how does X work / where is Y decided" from it, and check the spec against the code for gaps. Use when the user wants to understand a project or its design, before building a step, or to check whether the spec still matches the code.
---

# Understand a spec

Two jobs: explain the spec, and check it against the code. Neither changes anything; fixes are made after the user decides (spec changes through `spec-update`).

Specs written with `spec-write` follow its template, but older specs may not: go by the headings in the spec itself, never by the template's section numbers.

## Explain

- **Top-down, never a dump.** Goal first, then the whole flow in one line, then each module's one-line flow. Go into a module's detail only when asked.
- **Answer from the spec with references:** the section (SPEC §3.2) and the code (`file:line`), and say who makes the decision: code, the LLM, the user, or another model or service.
- **If the spec does not answer it,** say so, answer from the code, and note the gap.
- **Before building a step,** read the goal, the flow, that step's module section and its lines in the decisions log.

## Check against the code

**Scope:** the working tree (uncommitted changes included) of the app's code, its config files and its prompts. Tests, evals and the frontend only if the user asks.

**A module is covered** when the spec describes what it does: its own section, a row in a table, or a paragraph, even without naming the class or file.

**A one-line flow** is a single line at the start of a module's section showing its path, each step marked with who does it:
`triage_ticket called → load ticket (code) → pick category (LLM) → check it is allowed (code) → return`

Look for:

1. **Code not covered by the spec:** every tool, graph node, processing class or function module, the API, and anything else that makes a decision.
2. **Spec sections for things the code no longer has** (removed or renamed), and decisions-log lines that later changes contradict.
3. **Module sections without a one-line flow.**
4. **Places where the code differs from the spec:** a different order of steps, a different number (compare every threshold, limit and timeout in the spec with the config and with numbers hard-coded in the code), or a different decider (the spec says code, the code asks the LLM).
5. **Decisions in the code the spec does not mention:** a rule that rejects, routes or limits something, or a config value chosen for a reason. Skip comments that only explain how the code works.

**Rank the gaps** by harm: first a wrong result the spec promises cannot happen, then behaviour that differs, then missing documentation.

Report them as a table and fix nothing:

| Gap | Spec says | Code does | Fix in | Suggested fix |
|---|---|---|---|---|
| Unknown category goes to the general queue | §4: "a category outside the list → needs_review" | `graph/nodes.py:52` falls back to `"general"` | code | send it to `needs_review`; add a test |
| Refund limit is 1000 in code | §6: "over $500 goes to a person" | `config.yaml:12` `refund_limit: 1000` | decide | which number is right? |

**Fix in** is `spec` (the code is right, the spec is out of date), `code` (the spec is right, the code is wrong) or `decide` (the user has to choose).

To test the user's understanding afterwards, offer `repo-quiz`.
