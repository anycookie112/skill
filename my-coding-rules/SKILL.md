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
4. **Always start with a spec; it is the source of truth.** `spec-write` for a new one; `spec-update` whenever the design changes or a request is not in the spec (flag it before any code); `spec-understand` to read it before building a step; `spec-gap` to check it against the code.
5. **Commit only when the user asks.** Never commit `.env` or any secret.

## Code

- **Each file starts with a docstring** saying what it is for; where a file reads `.env` variables, the docstring lists them.
- **Validate data where it enters, then trust the types.** Anything from outside (API requests, LLM output, files) is parsed once at the boundary into a typed model: Pydantic, or a `TypedDict` for graph state. No `Any` in function signatures, no `dict[str, Any]` or bare `dict` for data with known fields (give it a model), and no `isinstance` checks scattered through the logic.
- **Every type override says why.** Each `cast(...)` and `# type: ignore[<code>]` has a comment right above it saying why it is safe.
- **Tests follow the `testing` skill:** a test that runs an AI workflow writes a run log of the flow and every LLM and tool call.
- **Run the linter before saying a change is done.** `uv run ruff check` must pass (settings in `pyproject.toml`, see `repo-layout`). Fix the code. Silence a rule only where it truly does not fit (for example a logger that records values of any type), with `# noqa: <code>` and the reason on the same line: `# noqa: ANN401 - logs whatever LangChain passes`.

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
