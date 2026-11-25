---
name: spec-understand
description: Read and explain a project's spec (SPEC.md) top-down, answer "how does X work / where is Y decided" from it, and check the spec against the code for gaps. Use when the user wants to understand a project or its design, before building a step, or to check whether the spec still matches the code.
---

# Understand a spec

Two jobs: explain the spec, and check it against the code. Neither changes anything; fixes go through `spec-update` once the user agrees.

## Explain

- **Top-down, never a dump.** Goal first, then the whole flow in one line, then each module's one-line flow. Go into a module's detail only when asked.
- **Answer from the spec with references:** the section (SPEC §3.5.1) and the code (`file:line`), and say who makes the decision: code, LLM, Jev, or the user.
- **If the spec does not answer it,** say so, answer from the code, and note the gap for `spec-update`.
- **Before building a step,** read the goal, the flow, that step's module section and its lines in the decisions log.

## Check against the code

Look for:

1. **Modules in the code with no spec section:** each tool in `agent/tools/`, each node in `graph/`, each class in `utils/` (see `repo-layout`).
2. **Spec sections for things the code no longer has** (removed or renamed).
3. **Module sections without a one-line flow.**
4. **Places where the code differs from the spec:** a different order of steps, a different threshold (compare `config.yaml` with the numbers in the spec), or a different decider (the spec says code, the code asks the LLM).
5. **Decisions in the code that the spec does not mention:** a "why" comment or a config value with no spec section or decisions-log line.

Report the gaps as a table, most important first, and fix nothing:

| Gap | Spec | Code | Suggested fix |
|---|---|---|---|
| `get_person` has no section | none | `agent/tools/get_person.py` | add a §3.5 module section |

To test the user's understanding afterwards, offer `repo-quiz`.
