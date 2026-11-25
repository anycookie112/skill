# skill

My Claude Code skills: the rules and the repo layout I want AI coding agents to follow when they write code for me.

These are my preferences for my own projects (Python LLM / agent apps built with LangChain, LangGraph and FastAPI), not a universal standard. If you use them, copy them and change them to match how you work.

## Install

Clone the repo as your Claude Code skills folder (only when `~/.claude/skills` does not exist yet):

```bash
git clone https://github.com/anycookie112/skill ~/.claude/skills
```

If you already have a skills folder, clone the repo somewhere else and copy the skill folders (`my-coding-rules/`, `repo-layout/`, `spec-write/`, `spec-update/`, `spec-understand/`) into `~/.claude/skills/`.

### Make it always on

A skill only loads when the agent decides it is relevant. To load these before every code change, add this line to `~/.claude/CLAUDE.md`, which Claude Code reads in every project:

```markdown
- Before writing or changing code in any project, load the `my-coding-rules` skill (how to work and write code) and the `repo-layout` skill (where things go), and follow both.
```

You can also load one by hand at any time: `/my-coding-rules` or `/repo-layout`.

### Update

```bash
cd ~/.claude/skills
git pull
```

## Skills

### `my-coding-rules`: what to do

- **How to work:** only write what is asked; build in small runnable steps; simple code I can understand line by line; commit only when asked, never `.env`.
- **Spec first:** every project starts with a spec, and the spec is the source of truth, through the three `spec-` skills below.
- **Code:** every file starts with a docstring, listing the `.env` variables it reads. Data from outside is validated once where it enters, into typed models; no `Any` and no loose dicts after that. Every `cast` or `# type: ignore` says why it is safe. `ruff check` passes before a change is done; a rule is silenced only with the reason on the same line.
- **Agents and LangChain / LangGraph:** one main `Agent` class to inherit from; library built-ins over hand-written loops; tools update the graph state with `Command`; the LLM provider is switched through `.env`.
- **No hard-coded matching rules:** no keyword lists or name-variant tables. Tools return candidates, the LLM judges the ambiguous cases, and code decides only exact identifiers and arithmetic.
- **Privacy:** ID numbers are masked (last 4 characters) before they reach the model, the state, the logs or API responses.

### `repo-layout`: where it goes

The folder tree every repo follows (root docs, `src/frontend`, `src/backend` with `graph/`, `agent/`, `agent/tools/`, `utils/`, then `tests/`, `evals/`, `logs/`) and where secrets, tunable values, per-machine settings, business logic, external data and docs belong. Also the standard Ruff settings that check some of the rules automatically.

### `spec-write`: start a spec

Writes `SPEC.md` for a new project: surface view first (goal, out of scope, stack, flow, steps), every module as a one-line flow, detail only for the next step. Holds the spec template.

### `spec-update`: keep it true

Flags any request that is not in the spec before code is written. Fills in a module's detail when its step comes up, changes only the section that changed, and adds a decisions-log line, in the same change as the code.

### `spec-understand`: read and check it

Explains a spec top-down (goal, flow, module flows, detail on request) with section and `file:line` references. Checks the spec against the code and reports the gaps as a table without fixing them.

## Examples

Each snippet below breaks a rule; the one after it is what the rule asks for.

### Spec: one-line flow per module

Rejected: a module section that starts with paragraphs, so the path through it is hidden.

Instead, open the section with its flow, each step marked with who does it:

```markdown
#### Watchlist screening

`search_watchlist called → ID join (code) → name shortlist (code) → exact name (code), else Jev → outcome (code) → return`

1. ID join over the whole watchlist (code) ...
2. Name shortlist (code) ...

**Why it's done this way**
- ...
```

### Spec: flag what is not in it

Asked: *"Add a CSV export of the results."* The spec has no export.

Rejected: writing the endpoint.

Instead: *"CSV export is not in the spec. Should I add it to the spec (§3.7 API) first?"* The spec and its decisions log are updated, then the code.

### No hard-coded matching rules

```python
SUFFIXES = ["sdn bhd", "sdn. bhd.", "bhd", "ltd"]

def same_company(a: str, b: str) -> bool:
    strip = lambda s: next((s[: -len(x)] for x in SUFFIXES if s.lower().endswith(x)), s)
    return strip(a).strip().lower() == strip(b).strip().lower()
```

Instead, code finds candidates and a model judges them:

```python
candidates = process.extract(holder_name, registered_names, scorer=fuzz.WRatio, limit=3)  # ranks, never decides
# the LLM (or Jev) judges whether each candidate is the same company
```

Exact identifier comparisons and arithmetic stay in code.

### The LLM provider is switchable

```python
llm = ChatAnthropic(model="claude-haiku-5-5")
```

Instead:

```python
llm = get_llm()  # provider and model come from .env (LLM_PROVIDER, LLM_MODEL)
```

### Tools update the graph state

```python
@tool
def search_watchlist(party_ids: list[str]) -> str:
    results = screener.screen(parties)
    return json.dumps(results)  # the model sees it, the graph state never does
```

Instead, return a `Command` with the state change and the message for the model:

```python
return Command(update={
    "watchlist_results": {r.party_id: r.model_dump() for r in results},
    "messages": [ToolMessage(json.dumps(reply), tool_call_id=tool_call_id)],
})
```

### Mask ID numbers

```python
log.info("Screening %s (ID %s)", person["full_name"], person["id_number"])
```

Instead:

```python
log.info("Screening %s (ID %s)", person["full_name"], mask_id(person["id_number"]))  # ****5642
```

### Validate where data enters

```python
def extract_fields(raw: Any) -> dict:
    if isinstance(raw, dict) and "closing_balance" in raw:
        ...
```

Instead, parse once at the boundary and pass the model on:

```python
fields = llm.with_structured_output(StatementFields).invoke(prompt)  # a Pydantic model from here on
```

### Every type override says why

```python
company = cast(Company, record)
```

Instead:

```python
# Safe: get_company only returns records it read from companies.json, which all have these fields.
company = cast(Company, record)
```

### No business logic in `main.py`

```python
@app.post("/runs")
async def create_run(file: UploadFile):
    data = await file.read()
    with zipfile.ZipFile(io.BytesIO(data)) as z:  # unzipping and checking files inside the route
        ...
```

Instead, the route calls the pipeline, and the processing lives in `utils/` and `graph/`.
