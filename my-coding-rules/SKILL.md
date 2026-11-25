---
name: my-coding-rules
description: The user's working rules and code conventions. Use before writing or changing code in any of their projects, especially Python LLM / agent apps built with LangChain, LangGraph and FastAPI - how to work, how specs are written, and how agents, tools and graphs must be built.
---

# My coding rules

Follow these for every change in the user's projects.

## How to work

1. **Only write what the user asks for.** Do not add modules, packages or helpers on your own initiative, even when the requested code would use them. If a requested piece needs something that does not exist yet, stop, say what it needs, and ask how they want it.
2. **Go step by step and explain.** Build in small pieces the user can run and see working before the next piece. When a concept is new to them, explain it in terms of what they already know.
3. **The user must understand every line**, so prefer simple, standard code over clever code.
4. **Always start with a spec; it is the source of truth.**
   - **Surface view first:** a top-down overview of what it does, the main parts, the stack (frameworks, LLM provider, storage) and the flow between parts. **Never overplan:** detail is written only for the step about to be built.
   - **Each module gets its own section** (a tool, a graph node, a processing class), starting with its flow in one line, each step marked with who does it (code, LLM, Jev, ...):
     `search_watchlist called → ID join (code) → name shortlist (code) → exact name (code), else Jev → outcome (code) → return`
     Then the steps in order, then "Why it's done this way".
   - **Flag anything that is not in the spec.** If a request is not covered by the spec, or goes against it, say so before writing any code: name what is new or what it conflicts with, and ask whether to add it to the spec first. Do not build it silently.
   - **Read the spec before building a step.** When the design changes, update the spec in the same change and add a line to its decisions log (date | decision | why).
5. **Commit only when the user asks.** Never commit `.env` or any secret.

## Code

- **Each file starts with a docstring** saying what it is for; where a file reads `.env` variables, the docstring lists them.

## Agents and LangChain / LangGraph

- **Agents:** one main `Agent` class; every other agent inherits from it and only sets what is specific to it (prompt, tools, when it is done).
- **Use LangChain/LangGraph built-ins instead of re-implementing them**: `ToolNode` and conditional edges for the agent loop, `@tool` / `StructuredTool` for tools, `Command` for state updates, reducers for merged state keys. Do not hand-write what the library already provides.
- **Tool calls update the graph state**: a tool returns `Command(update={...})` with its state changes plus the `ToolMessage` for the model (`InjectedToolCallId`, `InjectedState` where needed).
- **The LLM provider is switchable** through `.env` via `get_llm()`; never hard-code a provider or model in the logic.

## No hard-coded matching rules

- **Do not hard-code keyword lists or word-based matching rules** (e.g. lists of suffixes, abbreviation tables, name-variant rules). Ambiguous judgements such as name variants and abbreviations are made by the LLM.
- Tools may look up or rank data (e.g. fuzzy similarity with a library's built-in normaliser), but they return candidates; they do not decide what counts as a match.
- Decisions that are certain stay in code: exact identifier comparisons and all arithmetic.

## Data and privacy

- Mask ID numbers and similar identifiers (last 4 characters visible) before they reach the model, the state, logs or API responses.
