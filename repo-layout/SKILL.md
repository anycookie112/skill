---
name: repo-layout
description: How the user's repos are laid out - which folders and root files exist and what goes in each. Use when starting a new repo, adding a file or folder, or deciding where a piece of code, config, test, doc or data belongs. Taken from the seedflex repo (Python, FastAPI, LangGraph, uv).
---

# Repo layout

Every repo follows this layout. Put new code where its kind already lives; if nothing fits, ask before creating a new folder.

## The tree

```
<repo>/
  README.md                what it is, how to set it up and run it
  SPEC.md                  the spec: source of truth for the design, with a decisions log
  EVALS.md                 how the app is evaluated and the latest results (LLM apps)
  AI_USAGE.md              how AI tools were used to build it (when asked for)
  pyproject.toml           project, dependencies, scripts, pytest and ruff settings (uv)
  uv.lock                  locked versions (uv writes it; commit it)
  requirements.txt         exported from uv, for tools that need pip
  .python-version
  .env.example             every .env variable, with placeholder values (commit this, never .env)
  .gitignore  .dockerignore  Dockerfile

  src/
    frontend/              simple frontend calling the API (a single index.html is fine)
    backend/               the Python package (the import name: `from backend.x import y`)
      main.py              API (FastAPI): routes only, thin, no business logic
      llm.py               get_llm(): the LLM provider switch, read from .env
      settings.py          values from .env (paths, flags), each behind a small function
      config.yaml          tunable values: thresholds, timeouts, model options, prices
      graph/               the LangGraph pipeline
        state.py           the state TypedDict and its reducers
        nodes.py           the node functions
        build.py           wires nodes and edges, compiles the graph
      agent/
        agent.py           Agent: the main agent class
        <name>_agent.py    each agent inherits from Agent
        prompts.yaml       system prompts
        tools/             one file per tool function, named after the tool
      utils/               processing classes, one per file (extraction, OCR, matching, masking, logging)

  tests/
    conftest.py            shared fixtures
    test_<behaviour>.py    one file per behaviour being tested (test_login.py, test_verdict.py)

  evals/
    run_eval.py            runs the eval cases through the real app and scores them
    results/<timestamp>.json   one file per eval run (committed, so results can be compared)

  logs/                    runtime logs (git-ignored)
```

## Where things go

- **Secrets** (API keys, passwords) only in `.env`. Every variable is listed in `.env.example` with a placeholder.
- **Values someone might tune** (thresholds, timeouts, model options) go in `config.yaml` with a comment saying what each does. Nothing tunable is hard-coded in the logic.
- **Values that differ per machine** (data folder, port, password) go in `.env` and are read in `settings.py` or `llm.py`.
- **Business logic** goes in `utils/`, `graph/` and `agent/`, never in `main.py`.
- **Data that isn't part of the project** (provided data packs, briefs, drafts) stays out of git through `.gitignore`. The code reaches it through a path in `.env`.
- **Docs live at the root** as `UPPERCASE.md`. Do not start a `docs/` folder unless asked.
- **Generated or local files** are git-ignored: `.venv`, `__pycache__/`, `logs/`, `*.db`, `.env`.

## Linter (Ruff)

Every repo has Ruff as a dev dependency (`uv add --dev ruff`) and this in `pyproject.toml`:

```toml
[tool.ruff.lint]
extend-select = [
    "D100",    # every module starts with a docstring
    "D104",    # every package (__init__.py) starts with a docstring
    "ANN401",  # no Any in function arguments or return types
    "PGH003",  # a "# type: ignore" must name the error it ignores
]
```

Run it with `uv run ruff check`.
