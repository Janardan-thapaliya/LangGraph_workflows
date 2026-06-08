# LangGraph Workflows

Hands-on notebooks exploring how to build, structure, and orchestrate LangGraph `StateGraph` pipelines — from simple sequential chains to parallel fan-out, conditional branching, iterative loops, and stateful chatbots.

[![Open LangGraph_workflows In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/LangGraph_workflows.ipynb)
[![Open ChatBot_Workflow In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/ChatBot_Workflow.ipynb)

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

### 1. Sequential Workflow — BMI Calculator

The simplest pattern: nodes execute one after another in a straight chain.

```
START → calculate_bmi → label_bmi → END
```

- **State:** `weight_kg`, `height_m`, `bmi`, `category`
- `calculate_bmi` computes `bmi = weight / height²`
- `label_bmi` classifies into Underweight / Normal / Overweight / Obese

> 📸 *Place graph image here — run `g.get_graph().draw_mermaid_png()` in the notebook and save the output.*

---

### 2. LLM Workflow — Single-Node QA

Wires a `ChatOpenAI` call directly into the graph as a single node.

```
START → llm_qa → END
```

- **State:** `question`, `answer`
- `llm_qa` reads `question` from state, invokes the LLM, writes `answer` back

> 📸 *Place graph image here.*

---

### 3. Prompt Chaining — Blog Generator

Decomposes a task into a sequence of LLM calls, each building on the previous output.

```
START → get_outline → gen_blog → score_blog → END
```

- **State:** `topic`, `outline`, `answer`, `score`
- `get_outline` — generates a structured outline for the given topic
- `gen_blog` — writes the full blog post from the outline
- `score_blog` — rates the blog quality (1–10, integer) against the outline

> 📸 *Place graph image here.*

---

### 4. Parallel Workflow — Cricket Stats

Fans out from a single node into multiple independent branches that execute concurrently, then merges results.

```
START → input → calc_sr  ─┐
                calc_bpb ─┤→ display → END
                calc_bpf ─┘
```

- **State:** `runs`, `balls`, `fours`, `sixes`, `sr`, `bpb`, `bpf`
- `calc_sr` — strike rate = `(runs / balls) * 100`
- `calc_bpb` — balls per boundary
- `calc_bpf` — boundary frequency
- All three run in parallel; `display` aggregates and prints the stats

> 📸 *Place graph image here.*

---

### 5. Conditional Workflow — Grade Classifier

Routes execution to different nodes based on a condition evaluated at runtime.

```
START → grade → router ──→ pass_message → END
                       └──→ fail_message → END
```

- **State:** `score`, `grade`, `message`
- `grade` converts numeric score to letter grade (A / B / C / D / F)
- `router` uses `add_conditional_edges` to branch on pass/fail

> 📸 *Place graph image here.*

---

### 6. Iterative Workflow — Countdown Loop

Demonstrates a loop that repeatedly invokes a node until a termination condition is met.

```
START → countdown → should_continue? ──→ countdown (loop)
                                    └──→ END
```

- **State:** `count`
- Loop continues while `count > 0`, decrementing each iteration
- Uses `add_conditional_edges` with a router function to decide loop vs. exit

> 📸 *Place graph image here.*

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

> 📸 *Place a screenshot of a multi-turn conversation here to show memory in action.*

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
