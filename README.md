# LangGraph Workflows

Hands-on notebooks exploring how to build, structure, and orchestrate LangGraph `StateGraph` pipelines — from simple sequential chains to parallel fan-out, conditional branching, iterative loops, and stateful chatbots.

Langgraph Workflow: [![Open LangGraph_workflows In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/LangGraph_workflows.ipynb)

ChatBot Workflow: [![Open ChatBot_Workflow In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/ChatBot_Workflow.ipynb)

---

## Core Concepts

Every LangGraph graph is built on three primitives:

| Primitive | Role |
|---|---|
| **State** (`TypedDict`) | Shared data store passed between nodes |
| **Nodes** (Python functions) | Receive state, perform work, return updated state |
| **Edges** (`add_edge` / `add_conditional_edges`) | Define execution order and routing logic |

```
StateGraph(MyState)
  .add_node('node_name', my_function)
  .add_edge(START, 'node_name')
  .add_edge('node_name', END)
  .compile()
  .invoke(initial_state)
```

---

## Notebook 1 — `LangGraph_workflows.ipynb`

Covers the four fundamental workflow patterns, each demonstrated with a self-contained example.

### 1. Sequential Workflow — Blog Generator (prompt chained)

Decomposes a task into a sequence of LLM calls, each building on the previous output.

- **State:** `topic`, `outline`, `answer`, `score`
- `get_outline` — generates a structured outline for the given topic
- `gen_blog` — writes the full blog post from the outline
- `score_blog` — rates the blog quality (1–10, integer) against the outline

<img width="141" height="478" alt="image" src="https://github.com/user-attachments/assets/d90ed2af-54a8-4f1a-8968-a335631be91d" />

---

### 2. Parallel Workflow — Cricket Stats

Fans out from a single node into multiple independent branches that execute concurrently, then merges results.

- **State:** `runs`, `balls`, `fours`, `sixes`, `sr`, `bpb`, `bpf`
- `calc_sr` — strike rate = `(runs / balls) * 100`
- `calc_bpb` — balls per boundary
- `calc_bpf` — boundary frequency
- All three run in parallel; `display` aggregates and prints the stats

<img width="560" height="372" alt="image" src="https://github.com/user-attachments/assets/5739edad-3b4f-4379-b386-19a48ea8d411" />

---

### 3. Conditional Workflow — Review Handling

Routes execution to different nodes based on a condition evaluated at runtime.

- **State:** `review`, `sentiment`, `diagnosis`, `reply`
- Get review as Input --> Find sentiment of the review using LLM
- If positive (+ve) --> Reply thanks and appreciation message
- If negative (-ve) --> Run diagnosis --> Use diagnosis result to create suitable reply
Diagnosis will have structured: issue_type, tone, urgency

<img width="367" height="478" alt="image" src="https://github.com/user-attachments/assets/cc91cce9-358b-41d5-be10-c64c78812eb3" />

---

### 4. Iterative Workflow — Funny tweet writer for X

Demonstrates a loop that repeatedly invokes a node until a termination condition is met.

- **State:** `topic`, `tweet`, `evaluation`, `evaluator_feedback`, `iteration`, `max_iteration`, `tweet_history`, `feedback_history`
- The task is to automate funny and original tweet for X platform.
- The topic will be provided.
- The generator will use it to generate tweet --> passed to evaluator to check quality.
- Condition will be checked if the quality is good enough --> If good, used for posting so END
- If bad, it will be iteratively Optimized until it passes the quality check

<img width="372" height="408" alt="image" src="https://github.com/user-attachments/assets/f3bef589-0f58-4cca-9e6e-80a95f354930" />

---

## Notebook 2 — `ChatBot_Workflow.ipynb`

Builds a stateful conversational chatbot using LangGraph's built-in `MessagesState`.

```
START → chatbot → END  (invoked repeatedly, state persists across turns)
```

- Uses `MessagesState` — a pre-built state that maintains a `messages` list (full conversation history)
- Each call appends the new human message, runs the LLM over the full history, and appends the AI response
- **Memory** is handled automatically by `MessagesState`; no manual message management needed
- Uses `ChatOpenAI` as the underlying LLM
- Tool calling capability is also used for limited task

<img width="251" height="276" alt="image" src="https://github.com/user-attachments/assets/beb65976-ed92-4e6e-adec-e2205bb0364d" />

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
| `langgraph` | `StateGraph`, `START`, `END`, `MessagesState` |
| `langchain_openai` | `ChatOpenAI` |
| `langchain` | Core abstractions |
| `python-dotenv` | API key management |
