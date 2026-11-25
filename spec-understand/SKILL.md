---
name: spec-understand
description: Read and explain a project's spec (SPEC.md) top-down, and answer "how does X work / where is Y decided" from it with section and file:line references. Use when the user wants to understand a project or its design, or before building a step.
---

# Understand a spec

Explain the spec; change nothing. To check whether the spec still matches the code, use `spec-gap`.

Specs written with `spec-write` follow its template, but older specs may not: go by the headings in the spec itself, never by the template's section numbers.

- **Top-down, never a dump.** Goal first, then the whole flow in one line, then each module's one-line flow. Go into a module's detail only when asked.
- **Answer from the spec with references:** the section (SPEC §3.2) and the code (`file:line`), and say who makes the decision: code, the LLM, the user, or another model or service.
- **If the spec does not answer it,** say so, answer from the code, and suggest `spec-gap` or `spec-update`.
- **Before building a step,** read the goal, the flow, that step's module section and its lines in the decisions log.

To test the user's understanding afterwards, offer `repo-quiz`.
