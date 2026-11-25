# skill

My Claude Code skills: the rules and the repo layout I want AI coding agents to follow when they write code for me.

These are my preferences for my own projects (Python LLM / agent apps built with LangChain, LangGraph and FastAPI), not a universal standard. If you use them, copy them and change them to match how you work.

## Install

Clone the repo as your Claude Code skills folder (only when `~/.claude/skills` does not exist yet):

```bash
git clone https://github.com/anycookie112/skill ~/.claude/skills
```

If you already have a skills folder, clone the repo somewhere else and copy the skill folders (`my-coding-rules/`, `repo-layout/`, `spec-write/`, `spec-update/`, `spec-understand/`, `spec-gap/`) into `~/.claude/skills/`.

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
- **Spec first:** every project starts with a spec, and the spec is the source of truth, through the four `spec-` skills below.
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

### `spec-understand`: read it

Explains a spec top-down (goal, flow, module flows, detail on request) and answers "how does X work" with section and `file:line` references.

### `spec-gap`: check it against the code

Finds where the spec and the code disagree: uncovered code, stale sections, missing flows, different numbers or steps, promises the code can break, unrecorded decisions. Reports them ranked by harm, each marked "fix in spec", "fix in code" or "decide". Also lists what the spec describes but is not built yet, as a to-do list. Fixes nothing.

## Examples

Each snippet below breaks a rule; the one after it is what the rule asks for. The examples use a made-up support-ticket assistant; the rules apply to any project.

### Spec: one-line flow per module

Rejected: a module section that starts with paragraphs, so the path through it is hidden.

Instead, open the section with its flow, each step marked with who does it:

```markdown
#### Ticket triage

`triage_ticket called → load ticket (code) → pick category (LLM) → check it is an allowed category (code) → route to a queue (code) → return`

1. Load the ticket (code) ...
2. Pick the category (LLM) ...

**Why it's done this way**
- ...
```

### Spec: flag what is not in it

Asked: *"Add a CSV export of the tickets."* The spec has no export.

Rejected: writing the endpoint.

Instead: *"CSV export is not in the spec. Should I add it to the API section first?"* The spec and its decisions log are updated, then the code.

### No hard-coded matching rules

```python
ABBREVIATIONS = {"st": "street", "rd": "road", "ave": "avenue"}

def same_address(a: str, b: str) -> bool:
    expand = lambda s: " ".join(ABBREVIATIONS.get(w, w) for w in s.lower().split())
    return expand(a) == expand(b)
```

Instead, code finds candidates and a model judges them:

```python
candidates = process.extract(address, known_addresses, scorer=fuzz.WRatio, limit=3)  # ranks, never decides
# the LLM judges whether any candidate is the same place
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
def lookup_customer(email: str) -> str:
    customer = crm.find(email)
    return customer.model_dump_json()  # the model sees it, the graph state never does
```

Instead, return a `Command` with the state change and the message for the model:

```python
return Command(update={
    "customer": customer.model_dump(),
    "messages": [ToolMessage(customer.model_dump_json(), tool_call_id=tool_call_id)],
})
```

### Mask ID numbers

```python
log.info("Refund for %s, card %s", customer.name, customer.card_number)
```

Instead:

```python
log.info("Refund for %s, card %s", customer.name, mask_id(customer.card_number))  # ************4242
```

### Validate where data enters

```python
def create_refund(payload: Any) -> dict:
    if isinstance(payload, dict) and "amount" in payload:
        ...
```

Instead, parse once at the boundary and pass the model on:

```python
class RefundRequest(BaseModel):
    order_id: str
    amount: Decimal

@app.post("/refunds")
def create_refund(request: RefundRequest) -> Refund:  # FastAPI validates it; a typed model from here on
    ...

category = llm.with_structured_output(TicketCategory).invoke(prompt)  # the same for LLM output
```

### Every type override says why

```python
order = cast(Order, row)
```

Instead:

```python
# Safe: the orders query selects every Order column, so each row has all the fields.
order = cast(Order, row)
```

### No business logic in `main.py`

```python
@app.post("/tickets")
async def create_ticket(ticket: NewTicket):
    category = llm.invoke(f"Classify: {ticket.body}").content  # the LLM call and the routing inside the route
    queue = "billing" if "refund" in category.lower() else "general"
    ...
```

Instead, the route calls the pipeline, and the processing lives in `utils/`, `graph/` and `agent/`.
