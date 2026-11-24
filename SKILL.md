---
name: my-coding-rules
description: The user's working rules and code conventions. Use before writing or changing code in any of their projects, especially Python LLM / agent apps built with LangChain, LangGraph and FastAPI - what to write, where it goes, and how agents, tools and graphs must be built.
---

# My coding rules

Follow these for every change in the user's projects.

## How to work

1. **Only write what the user asks for.** Do not add modules, packages or helpers on your own initiative, even when the requested code would use them. If a requested piece needs something that does not exist yet, stop, say what it needs, and ask how they want it.
2. **Go step by step and explain.** Build in small pieces the user can run and see working before the next piece. When a concept is new to them (for example LangGraph state, nodes, edges, reducers, `Command`), explain it in terms of what they already know (LangChain chains).
3. **The user must understand every line**, so prefer simple, standard code over clever code.
4. **If the project has a design doc or spec, it is the source of truth.** Read it before building a step, and record design changes in it.
5. **Commit only when the user asks.** Never commit `.env` or any secret.

## Code structure (LLM / agent apps)

```
src/
  frontend/                    simple frontend calling the API
  backend/
    main.py                    API (FastAPI): thin, no business logic
    llm.py                     get_llm(): provider switch from .env
    graph/                     LangGraph pipeline (state, nodes, build)
    agent/
      agent.py                 Agent: the main agent class
      <name>_agent.py          each agent inherits from Agent
      prompts.yaml             system prompts
      tools/                   ONE FILE PER TOOL FUNCTION
    utils/                     processing classes (e.g. document extraction)
```

- **Agents:** one main `Agent` class; every other agent inherits from it and only sets what is specific to it (prompt, tools, when it is done).
- **Tools:** one file per tool in `agent/tools/`; agents import them from there.
- **Processing steps** (e.g. unzip, PDF to text) are classes in `utils/`.

## LangChain / LangGraph

- **Use LangChain/LangGraph built-ins instead of re-implementing them**: `ToolNode` and conditional edges for the agent loop, `@tool` / `StructuredTool` for tools, `Command` for state updates, reducers for merged state keys. Do not hand-write what the library already provides.
- **Tool calls update the graph state**: a tool returns `Command(update={...})` with its state changes plus the `ToolMessage` for the model (`InjectedToolCallId`, `InjectedState` where needed).
- **The LLM provider is switchable** through `.env` via `get_llm()`; never hard-code a provider or model in the logic.

## No hard-coded matching rules

- **Do not hard-code keyword lists or word-based matching rules** (e.g. lists of suffixes, abbreviation tables, name-variant rules). Ambiguous judgements such as name variants and abbreviations are made by the LLM.
- Tools may look up or rank data (e.g. fuzzy similarity with a library's built-in normaliser), but they return candidates; they do not decide what counts as a match.
- Decisions that are certain stay in code: exact identifier comparisons and all arithmetic.

## Data and privacy

- Mask ID numbers and similar identifiers (last 4 characters visible) before they reach the model, the state, logs or API responses.
