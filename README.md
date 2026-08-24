# Deep Research

A LangGraph-based multi-agent research system inspired by the original LangChain "Deep Research from Scratch" pattern, but implemented in this repository with a simpler project structure and a notebook-first workflow.

This project turns a user question into:

- a clarification/scoping step
- a structured research brief
- parallel research tasks delegated to specialized researcher agents
- a supervisor that coordinates the work
- a final synthesized report

---

## What this project does

The system follows a research pipeline:

1. User input is reviewed and clarified.
2. A research brief is generated from the conversation history.
3. A supervisor agent decides which subtopics to investigate.
4. Multiple researcher agents perform web searches and iterative reasoning.
5. Findings are compressed and returned to the supervisor.
6. A final report is generated from all collected research notes.

This is a good example of a LangGraph workflow with:

- stateful agent orchestration
- tool calling
- multi-agent delegation
- checkpointing with in-memory persistence
- notebook-driven experimentation

---

## Architecture

The repo is organized around a few key components:

### 1. Scoping workflow

Located in:

- `src/scoping/research_agent_scope.py`
- `src/scoping/state_scope.py`

This part determines whether the user request is clear enough and writes a research brief that becomes the foundation for the rest of the workflow.

### 2. Research agent

Located in:

- `src/research_agent/research_agent.py`
- `src/research_agent/state_research.py`

This is a specialized research worker that:

- uses Tavily search tools
- makes iterative tool calls when needed
- compresses findings into a summary for the supervisor

### 3. Supervisor

Located in:

- `src/research_supervisor/multi_agent_supervisor.py`
- `src/research_supervisor/state_multi_agent_supervisor.py`

This orchestrator decides whether to:

- delegate research tasks
- run reflection using `think_tool`
- mark the task as complete

### 4. Full workflow

Located in:

- `src/full_agent/research_agent_full.py`

This compiles the full graph and wires together:

- clarification
- brief generation
- supervisor subgraph
- final report generation

---

## Project structure

```text
.
├── main.ipynb
├── pyproject.toml
├── README.md
├── src/
│   ├── __init__.py
│   ├── prompts.py
│   ├── utils.py
│   ├── full_agent/
│   │   └── research_agent_full.py
│   ├── research_agent/
│   │   ├── research_agent.py
│   │   └── state_research.py
│   ├── research_supervisor/
│   │   ├── multi_agent_supervisor.py
│   │   └── state_multi_agent_supervisor.py
│   └── scoping/
│       ├── research_agent_scope.py
│       └── state_scope.py
└── .venv/
```

---

## Core dependencies

The project uses:

- Python 3.12+
- LangGraph
- LangChain
- OpenAI model integrations
- Tavily search API
- IPython / notebook support
- dotenv

See `pyproject.toml` for the exact dependency list.

---

## Setup

### 1. Create a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2. Install dependencies

```bash
pip install -U pip
pip install -e .
```

or, if you are using the project as a standard Python package with uv:

```bash
uv sync
```

---

## Environment variables

This project expects API keys in your environment. The repo loads variables from a local `.env` file via `src/__init__.py`.

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key
```

If the environment is configured correctly, the app will load them automatically when the package is imported.

---

## Running the app

The notebook at `main.ipynb` is the easiest entry point for using the full workflow.

Example usage from the notebook:

```python
from langchain_core.messages import HumanMessage

thread = {"configurable": {"thread_id": "1"}}

result = await agent.ainvoke(
    {
        "messages": [
            HumanMessage(content="I want to research the best coffee shops in Bhubaneswar.")
        ]
    },
    config=thread,
)
```

This pattern is important: the supervisor and other nodes are asynchronous, so use `ainvoke(...)` rather than `invoke(...)`.

---

## Important note on async calls

The project has async nodes such as:

- `supervisor`
- `supervisor_tools`
- final report generation logic

Because those are async, calling them with the synchronous `invoke` method can fail with:

```python
TypeError: No synchronous function provided to "supervisor".
```

Use:

```python
await agent.ainvoke(...)
```

instead of:

```python
agent.invoke(...)
```

---

## Recommended usage flow

### Notebook workflow

1. Open `main.ipynb`
2. Import the main agent from `src.full_agent.research_agent_full`
3. Start a thread config
4. Send a research question as a `HumanMessage`
5. Inspect the returned message history and final report

### Example

```python
from src.full_agent.research_agent_full import agent
from langchain_core.messages import HumanMessage

thread = {"configurable": {"thread_id": "demo-1"}}

response = await agent.ainvoke(
    {
        "messages": [
            HumanMessage(content="Compare the best AI research tools for enterprise teams.")
        ]
    },
    config=thread,
)

print(response["final_report"])
```

---

## Notes

This repository is a compact, educational implementation of a deep-research workflow. It is designed to show the underlying mechanics of:

- research scoping
- agent delegation
- tool-driven search
- iterative synthesis
- final report generation

It is based on the ideas from the original LangChain deep research-from-scratch project, but adapted to this codebase and project layout.

---

## License

This project does not currently declare a license file in the repo. Please check the repository status before publishing or sharing it externally.

---

## Troubleshooting

### Missing API keys

If the model or Tavily search fails, confirm your environment has:

```bash
echo $OPENAI_API_KEY
echo $TAVILY_API_KEY
```

### Async call error

If you see a `TypeError` about a missing synchronous function, switch from `invoke` to `ainvoke`.

### Search tool issues

If Tavily results are empty or errors appear, verify that:

- your `TAVILY_API_KEY` is valid
- the query is not blocked or malformed
- the network is available

---

## Summary

This repo is a practical LangGraph deep research workflow that combines:

- a scoping layer
- multiple research agents
- a coordinator/supervisor
- web search via Tavily
- synthesis into a polished final answer

It is a solid starting point for experimenting with autonomous research systems in Python.
