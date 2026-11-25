---
name: spec-write
description: Write a new spec (SPEC.md) for a new repo or a large new part of one - surface view first, one section per module with a one-line flow, never overplanned. Use at the start of a project, before any code. Holds the spec template that spec-update and spec-understand also use.
---

# Write a spec

The spec is the source of truth for the design. Write it before any code, and write only the surface view: what someone needs to see the whole thing from the top down. Detail is added later, one step at a time, with `spec-update`.

For a feature in a repo that already has a spec, do not start a new one: use `spec-update`.

## Steps

1. **Understand first.** Read any brief or data you were given. Ask what it is for, who uses it, and what is fixed (data, deadline, stack). Do not write until the goal is clear.
2. **Write the surface version** in `SPEC.md` at the repo root, from the template below:
   - sections 1 to 4 in full (goal, out of scope, stack, flow);
   - section 5 with every module as its name and its one-line flow only;
   - section 8 with every step as one line, and only the first one or two in detail;
   - sections 9 and 10 with the open questions and the first decisions (the stack choices and why).
3. **Show it and stop.** Wait for the user's OK. No code before it.
4. **Detail later.** When a step is about to be built, its module section is filled in with `spec-update`.

## Writing rules

- **Plain and short.** The surface view fits on about one page.
- **Mark who does each step:** (code), (LLM), (Jev), (user). Certain decisions are code; judgements are the LLM's.
- **Do not invent.** Anything unknown goes in Open questions, not in the design.
- **Do not overplan.** No function signatures, field lists or edge cases for steps that are not next.

## Template

```markdown
# <Project>

## 1. Goal
What it does and for whom, in 2-3 sentences.

## 2. Out of scope
What it deliberately does not do.

## 3. Stack
Language, frameworks, LLM provider, storage, hosting - one line each, with why if it was a choice.

## 4. Flow
The whole path in one line, then one line per part:
`input → part → part → output`

## 5. Modules
One section per tool, graph node or processing class.

### 5.1 <module>
`<module> called → step (code) → step (LLM) → return`

1. The steps in order, each marked with who does it.

**Why it's done this way**
- The reasons a reader would otherwise question.

## 6. Rules vs LLM
Which decisions are made in code and which are LLM judgement.

## 7. Failure modes
| What goes wrong | What happens |
|---|---|

## 8. Steps
The build order. Only the next one or two have detail; the rest stay one line.
1. <step> - done when <check>

## 9. Open questions
- <question> - who answers it

## 10. Decisions log
| Date | Decision | Why |
|---|---|---|
```

Add these sections only when the project needs them: API (endpoints, requests, responses), Storage, Acceptance criteria, Eval plan.

## Example module section

From the seedflex spec, once its step was built:

```markdown
#### Watchlist screening

`search_watchlist called → ID join (code) → name shortlist (code) → exact name (code), else Jev → outcome (code) → return`

1. **ID join over the whole watchlist (code).** An entry with the same ID number is a hit.
2. **Name shortlist (code).** Entries whose name scores at least 65 (rapidfuzz); the score only picks what gets checked.
3. **Same name?** Exact (code), otherwise Jev judges it (probability ≥ 0.7).
4. **Outcome (code):** hit / review / ruled out / no match.

**Why it's done this way**
- The ID join runs over the whole watchlist, so a listed party using another name is still caught.
- A name match is never a hit: the same name could be two people.
```
