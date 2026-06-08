# Chatbot Workflow

Builds a stateful conversational chatbot from scratch using LangGraph's `StateGraph`, progressing from a stateless skeleton to a fully persistent, memory-aware chatbot.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Janardan-thapaliya/LangGraph_workflows/blob/main/Chatbot%20Workflow/ChatBot_Workflow.ipynb)

---

## Where This Fits

```
StateGraph primitives  →  [LangGraph Workflows]  →  [Chatbot Workflow]  →  Advanced RAG
(sequential, parallel,      (core patterns)           (persistence,          (Self-RAG,
 conditional, iterative)                               memory, tools)          CRAG)
```

---

## What's Covered

### Phase 1 — Stateless Chatbot Skeleton

Builds the minimal viable LangGraph chatbot: a single `chat_node` that receives messages from state, calls the LLM, and returns the response.

```
START → chat_node → END
```

| Component | Detail |
|---|---|
| **State** | `ChatState(TypedDict)` with `messages: Annotated[list[BaseMessage], add_messages]` |
| **`add_messages`** | Reducer from `langgraph.graph.message` — appends new messages to the list instead of replacing it |
| **LLM** | `ChatOpenAI` (default: `gpt-3.5-turbo`) |
| **Invocation** | `g.invoke({'messages': [HumanMessage(...)]})` |

**Limitation discovered:** Each `invoke()` call starts fresh — no memory of prior turns. Demonstrated explicitly with a while loop that shows the bot forgetting previous messages.

---

### Phase 2 — Persistent Chatbot with `MemorySaver`

Solves the stateless problem by adding a **checkpointer** — an in-memory store that persists state across invocations by `thread_id`.

```
START → chat_node → END   (state persisted via MemorySaver per thread_id)
```

| Component | Detail |
|---|---|
| **Checkpointer** | `MemorySaver()` — stores full state snapshots in memory |
| **Compilation** | `graph.compile(checkpointer=checkpointer)` |
| **Thread config** | `config = {'configurable': {'thread_id': '1'}}` passed at invoke time |
| **Memory scope** | Each `thread_id` is an isolated conversation session |
| **State inspection** | `g.get_state(config=config)` returns a `StateSnapshot` with full message history |

**Result:** The bot remembers earlier messages within the same `thread_id`. Demonstrated with a multi-turn conversation where the bot correctly recalls `"Hi, I am Tom"` when asked `"What is my name?"` two turns later.

---

## Key Concepts

### `add_messages` vs plain list

`add_messages` (from `langgraph.graph.message`) behaves like `operator.add` but is optimised for chat — it deduplicates by message `id` and correctly handles message updates. Without it, returning `{'messages': [response]}` would overwrite the entire history.

### Why `MemorySaver` fixes the problem

Without a checkpointer, `invoke()` is stateless — it reads the input state and discards everything after returning. With `MemorySaver`, LangGraph automatically:
1. Loads the previous `StateSnapshot` for the given `thread_id` before execution
2. Merges the new input into the loaded state using the `add_messages` reducer
3. Saves the updated state back to the checkpointer after execution

---

## Setup

```bash
pip install langgraph langchain langchain_openai langchain_core python-dotenv
```

Create a `.env` file:

```env
OPENAI_API_KEY=your_openai_key
```

---

## Dependencies

| Package | Purpose |
|---|---|
| `langgraph` | `StateGraph`, `START`, `END`, `MemorySaver` |
| `langgraph.graph.message` | `add_messages` reducer |
| `langchain_openai` | `ChatOpenAI` |
| `langchain_core.messages` | `HumanMessage`, `BaseMessage` |
| `python-dotenv` | API key management |
