---
name: testing
description: How tests are written in the user's projects, especially tests that run an AI workflow (a LangGraph graph, an agent, tool calls) - each must write a run log of the flow and every LLM and tool call to tests/logs/. Use when writing or changing tests, or when a test of an AI workflow fails and you need to see what happened.
---

# Testing

## AI workflow tests write a run log

Every test that runs a graph or an agent writes a log file, `tests/logs/<test name>.jsonl`, overwritten on each run. When such a test fails, read its log to see what happened; do not guess.

One JSON object per line, one line per event:

| `event` | Written when | Fields |
|---|---|---|
| `node` | a graph node finished | `name` (its path: `triage/agent` for a node inside a subgraph), `ms`, `output` (the state keys it changed, summarised) |
| `llm` | an LLM call finished | `name` (the node it ran in), `ms`, `model`, `output` (the tool calls it asked for with their arguments, or its text) |
| `tool` | a tool call finished | `name`, `ms`, `input` (its arguments, without injected state), `output` (what the model was given back) |

Any event has `error` instead of `output` when it failed. Every line also has `time`. Tokens and cost are not needed.

Example, a made-up support-ticket assistant:

```
{"time": "...", "event": "node", "name": "load_ticket", "ms": 3, "output": {"ticket": "T-102"}}
{"time": "...", "event": "llm", "name": "triage/agent", "ms": 812, "model": "claude-haiku-5-5", "output": [{"name": "lookup_customer", "args": {"email": "a@example.com"}}]}
{"time": "...", "event": "tool", "name": "lookup_customer", "ms": 9, "input": {"email": "a@example.com"}, "output": "{\"plan\": \"enterprise\"}"}
{"time": "...", "event": "node", "name": "triage/tools", "ms": 10, "output": {"tool updates": [["customer", "messages"]]}}
{"time": "...", "event": "llm", "name": "triage/agent", "ms": 640, "model": "claude-haiku-5-5", "output": [{"name": "submit_category", "args": {"category": "billing"}}]}
{"time": "...", "event": "node", "name": "route", "ms": 0, "output": {"queue": "priority"}}
```

## How

- **One logger for the app and the tests.** The log is written by the app's run-log callback handler (a LangChain `BaseCallbackHandler`, passed as `config={"callbacks": [handler]}`), the same one the app uses in production. Tests do not have a logger of their own.
- **The handler is told where to write.** It takes the file or folder as an argument, so tests pass `tests/logs/` and the app passes `logs/`. Tests never write to the app's real `logs/` folder.
- **A shared fixture** in `tests/conftest.py` creates the handler for each test, named after the test (`request.node.name`).
- **Point to the log when a test fails:** the assertion message names the log file.
- **Keep it readable and safe:** long outputs are shortened; prompts, secrets and unmasked ID numbers are never logged.
- **`tests/logs/` is git-ignored.**
