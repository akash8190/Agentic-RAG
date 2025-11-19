# RAG-Tutorials

# Agentic-RAG

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> **Agentic-RAG** — A reference architecture and example implementation combining Retrieval-Augmented Generation (RAG) with an agentic orchestration layer. Built to demonstrate how autonomous agents can query knowledge stores, chain reasoning steps, and call tools while staying grounded on retrieved context.

---

## Table of contents

* [Project overview](#project-overview)
* [Why Agentic-RAG?](#why-agentic-rag)
* [Features](#features)
* [Architecture](#architecture)
* [Quick start](#quick-start)

  * [Requirements](#requirements)
  * [Install](#install)
  * [Run locally](#run-locally)
* [Configuration](#configuration)
* [Usage examples](#usage-examples)
* [Extending / adding tools](#extending--adding-tools)
* [Testing](#testing)
* [Contributing](#contributing)
* [License & credits](#license--credits)
* [Contact](#contact)

---

## Project overview

`Agentic-RAG` demonstrates a pattern for combining:

* **RAG (Retrieval-Augmented Generation)** — retrieve relevant documents from a vector store and provide them as grounding context to a language model.
* **Agentic control** — an orchestration layer that enables multi-step reasoning, tool calls (search, calculators, external APIs), and iterative retrieval.

This repo includes example code for building a local prototype using open-source vector stores, a lightweight agent controller, and a pluggable LLM backend.

## Why Agentic-RAG?

RAG alone helps LM outputs stay grounded, but complex tasks often need: multiple retrieval rounds, conditional tool use (e.g., calculators, web search), and explicit step management. An agentic layer enables this control flow while keeping generation grounded to retrieved evidence.

Use cases:

* Question answering over private corpora (docs, knowledge bases)
* Multi-step research assistants (summarization + citation)
* Automated workflows that must consult knowledge stores and external APIs

## Features

* Modular vector storage adapter (e.g., FAISS, Chroma)
* Retriever + contextual prompt builder for the LLM
* Agent controller that supports simple action/observation loops
* Example tool adapters (search, calculator, HTTP fetch)
* Example demo scripts and notebook-style walkthroughs

## Architecture

1. **Client**: user interfaces (CLI, notebook, web UI) that submit tasks.
2. **Agent Controller**: orchestrates steps — retrieve, plan, call tool, generate, loop.
3. **Retriever / Index**: vector DB + embeddings to fetch relevant passages.
4. **LLM Backend**: model interface for completion / chat.
5. **Tools**: external capabilities the agent can call (web, calc, DB query).

A simplified flow:

```
User -> Agent Controller
 Agent: retrieve(context) -> LLM(plan)
 If tool needed: Agent -> tool -> observation
 Agent: retrieve(more) -> LLM(final answer grounded on retrieved docs)
```

## Quick start

> These instructions assume a Python environment. Adjust if you prefer Node.js or another stack.

### Requirements

* Python 3.10+
* `pip` or `poetry`
* Optional: Docker (for containerized run)
* Optional: API keys for external LLM providers (if using hosted models)

### Install (pip)

```bash
# clone repo
git clone https://github.com/akash8190/Agentic-RAG.git
cd Agentic-RAG

# create venv
python -m venv .venv
source .venv/bin/activate  # on Windows: .venv\Scripts\activate

# install deps
pip install -r requirements.txt
```

If you use Poetry:

```bash
poetry install
poetry shell
```

### Run locally (example)

1. Create a `.env` file with required keys (see Configuration).
2. Build or load the index (if the repo contains `scripts/build_index.py`):

```bash
python scripts/build_index.py --data data/docs --index_path ./index
```

3. Run the demo agent:

```bash
python examples/agent_demo.py --index ./index
```

Open `notebooks/demo.ipynb` for a step-by-step walkthrough (if provided).

## Configuration

Set environment variables in `.env` or export in your shell. Example variables:

```
# LLM provider (optional)
OPENAI_API_KEY=sk-...
LLM_PROVIDER=openai

# Vector DB settings
VECTOR_STORE=faiss
EMBEDDING_MODEL=all-mpnet-base-v2

# Other toggles
AGENT_MAX_STEPS=6
RETRIEVAL_TOP_K=5
```

The project contains an abstraction layer so you can swap in other providers (e.g., local LLMs or cloud vector stores).

## Usage examples

### Simple Q&A (CLI)

```bash
python cli/qa.py --question "How does Agentic-RAG handle multi-step reasoning?"
```

Expected behavior:

* Agent retrieves top-K passages
* LLM formulates a plan; if plan contains actions, agent runs them
* Final answer returned with supporting references

### Programmatic (Python)

```py
from agentic_rag import Agent

agent = Agent(index_path="./index")
answer = agent.ask("Summarize the security considerations for deploying the system.")
print(answer)
```

## Extending / adding tools

Tools are small adapters that accept `action` inputs and return `observation`. To add a new tool:

1. Create a new file `tools/my_tool.py` implementing `run(action: dict) -> dict`.
2. Register it in the agent's tool registry (see `agentic_rag/tools/__init__.py`).
3. Update the agent policy prompt (if needed) to mention the tool's capabilities.

Example tool skeleton:

```py
# tools/calculator.py
class CalculatorTool:
    def run(self, expression: str) -> str:
        # safe eval wrapper or use an external math library
        return str(eval(expression, {"__builtins__":{}}))
```

## Testing

Run unit tests with pytest (if present):

```bash
pytest tests
```

Add tests for any new retrievers, tools, or orchestration logic you add.

## Contribution

Contributions — bug fixes, docs improvements, additional adapters — are welcome.

Suggested workflow:

1. Fork the repo
2. Create a feature branch
3. Add tests for new behavior
4. Open a Pull Request describing the change

Please follow the code style used across the project and keep functions small and well-documented.

## Design notes & caveats

* **Grounding**: always include retrieved passage identifiers or snippets with answers for traceability.
* **Safety**: be careful when enabling tools like `eval` or arbitrary HTTP requests; sandbox where possible.
* **Latency**: multi-step agentic loops can increase latency — consider caching or limiting steps for production.

## License & credits

This project is distributed under the MIT license. See `LICENSE` for details.

## Contact

Maintainer: Akash Kumar (GitHub: `akash8190`)

If you'd like, I can also:

* Add a `CONTRIBUTING.md` and issue templates
* Create example Dockerfile and GitHub Actions workflow
* Generate a short demo notebook with sample queries

---

*README generated for the repository `Agentic-RAG`. Feel free to edit, expand the examples, or request additional files (Dockerfile, CI, contributing guide).*
