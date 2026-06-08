# LangGraph Workflows

Hands-on notebooks exploring how to build, structure, and orchestrate LangGraph `StateGraph` pipelines — from simple sequential chains to parallel fan-out, conditional branching, iterative loops, and stateful chatbots.

---

## Repository Structure

```
LangGraph_workflows/
├── LangGraph_workflows.ipynb    ← Core workflow patterns (sequential, parallel, conditional, iterative)
├── Chatbot Workflow/            ← Stateless → persistent chatbot with MemorySaver
│   ├── ChatBot_Workflow.ipynb
│   └── readme.md
└── README.md
```

---

## Core Concepts

Every LangGraph graph is built on three primitives:

| Primitive | Role |
|---|---|
| **State** (`TypedDict`) | Shared data store passed between nodes |
| **Nodes** (Python functions) | Receive state, perform work, return updated state |
| **Edges** (`add_edge` / `add_conditional_edges`) | Define execution order and routing logic |

```python
StateGraph(MyState)
  .add_node('node_name', my_function)
  .add_edge(START, 'node_name')
  .add_edge('node_name', END)
  .compile()
  .invoke(initial_state)
```

---

## Notebook 1 — `LangGraph_workflows.ipynb`

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/LangGraph_workflows.ipynb)

Covers the four fundamental workflow patterns, each demonstrated with a self-contained example.

### 1. Sequential Workflow — Blog Generator

Decomposes a task into a sequence of LLM calls, each building on the previous output.

- **State:** `topic`, `outline`, `answer`, `score`
- `get_outline` → `gen_blog` → `score_blog`

<img width="141" height="478" alt="Sequential workflow graph" src="https://github.com/user-attachments/assets/d90ed2af-54a8-4f1a-8968-a335631be91d" />

---

### 2. Parallel Workflow — Cricket Stats

Fans out from a single node into multiple independent branches that execute concurrently, then merges results.

- **State:** `runs`, `balls`, `fours`, `sixes`, `sr`, `bpb`, `bpf`
- `calc_sr`, `calc_bpb`, `calc_bpf` run in parallel → `display` aggregates

<img width="560" height="372" alt="Parallel workflow graph" src="https://github.com/user-attachments/assets/5739edad-3b4f-4379-b386-19a48ea8d411" />

---

### 3. Conditional Workflow — Review Handling

Routes execution to different nodes based on a condition evaluated at runtime.

- **State:** `review`, `sentiment`, `diagnosis`, `reply`
- Positive review → thank-you reply; Negative review → diagnosis → crafted reply

<img width="367" height="478" alt="Conditional workflow graph" src="https://github.com/user-attachments/assets/cc91cce9-358b-41d5-be10-c64c78812eb3" />

---

### 4. Iterative Workflow — Funny Tweet Writer

Demonstrates a loop that repeatedly invokes a node until a quality threshold is met.

- **State:** `topic`, `tweet`, `evaluation`, `evaluator_feedback`, `iteration`, `max_iteration`, `tweet_history`, `feedback_history`
- `generator` → `evaluator` → conditional: pass → END, fail → loop back

<img width="372" height="408" alt="Iterative workflow graph" src="https://github.com/user-attachments/assets/f3bef589-0f58-4cca-9e6e-80a95f354930" />

---

## Notebook 2 — `Chatbot Workflow/ChatBot_Workflow.ipynb`

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/Chatbot%20Workflow/ChatBot_Workflow.ipynb)

Builds a stateful conversational chatbot, progressing from a stateless skeleton to a fully persistent chatbot using `MemorySaver`.

```
START → chat_node → END   (state persisted via MemorySaver per thread_id)
```

- **Phase 1:** Stateless skeleton — single `chat_node`, `add_messages` reducer, `ChatOpenAI`
- **Phase 2:** Persistent memory — `MemorySaver` checkpointer + `thread_id` config isolates conversation sessions
- `g.get_state(config)` returns a `StateSnapshot` for inspecting full message history

See [`Chatbot Workflow/readme.md`](./Chatbot%20Workflow/readme.md) for full documentation.

---

## Setup

```bash
pip install langgraph langchain langchain_openai python-dotenv
```

Create a `.env` file:

```env
OPENAI_API_KEY=your_openai_key
```

---

## Dependencies

| Package | Purpose |
|---|---|
| `langgraph` | `StateGraph`, `START`, `END`, `MemorySaver`, `MessagesState` |
| `langchain_openai` | `ChatOpenAI` |
| `langchain` | Core abstractions |
| `python-dotenv` | API key management |
