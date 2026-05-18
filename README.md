# LangGraph Agent Starter

A minimal, production-ready template for building agentic AI workflows using LangGraph — with conditional routing, tool integration, conversation memory, and a FastAPI interface.

Built from patterns used in real production deployments. Skip the boilerplate and start building agents that actually work.

---

## What This Gives You

- A working LangGraph state machine with conditional node routing
- Pluggable tool support (web search, APIs, custom functions)
- Conversation memory via Firestore or in-memory store
- FastAPI wrapper for serving the agent as a REST API
- Clean project structure that scales to production

---

## Architecture

```
User Message
     │
     ▼
┌─────────────────────────────────┐
│         LangGraph Agent         │
│                                 │
│  ┌──────────┐   ┌────────────┐  │
│  │  Router  │──▶│  Responder │  │
│  │   Node   │   │    Node    │  │
│  └──────────┘   └────────────┘  │
│       │                         │
│       ▼                         │
│  ┌──────────┐                   │
│  │  Tool    │                   │
│  │  Caller  │                   │
│  └──────────┘                   │
└─────────────────────────────────┘
     │
     ▼
FastAPI Response
```

**Router node** — classifies the incoming message and decides which path to take (direct response, tool call, or clarification).

**Responder node** — generates the final LLM response grounded in context and tool outputs.

**Tool caller node** — executes registered tools (search, APIs, custom functions) and returns results back into the graph state.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Orchestration | LangGraph |
| LLM APIs | Gemini · OpenAI · Anthropic |
| API framework | FastAPI |
| Memory | Firestore · In-memory |
| Embeddings | Sentence Transformers |
| Vector search | Qdrant |
| Deployment | GCP Cloud Run · Docker |

---

## Project Structure

```
langgraph-agent-starter/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── agent/
│   │   ├── graph.py         # LangGraph state machine definition
│   │   ├── nodes.py         # Router, responder, tool caller nodes
│   │   ├── state.py         # AgentState TypedDict schema
│   │   └── tools.py         # Tool definitions and registry
│   ├── memory/
│   │   └── store.py         # Firestore / in-memory conversation store
│   └── config.py            # Model, environment, and routing config
├── Dockerfile
├── requirements.txt
└── README.md
```

---

## Agent State Schema

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    user_id: str
    route: str          # "respond" | "tool_call" | "clarify"
    tool_output: str
    turn_count: int
```

---

## Routing Logic

The router node classifies each message and sets the `route` field in state:

```
incoming message
       │
       ├── factual / simple ──────▶ respond directly
       │
       ├── requires tool/search ──▶ tool_call ──▶ respond with result
       │
       └── ambiguous / unclear ───▶ clarify ──▶ ask follow-up
```

This keeps simple queries fast and cheap while routing complex ones to the right handler — without overloading a single prompt.

---

## Tiered LLM Pattern

```python
def get_model(complexity: str):
    if complexity == "simple":
        return gemini_flash_lite      # fast, ~$0.0001/query
    elif complexity == "complex":
        return gemini_25_flash        # capable, ~$0.001/query
```

Route by complexity, not by default. This pattern achieved ~$0.0003 average cost per interaction in production — roughly 30x under a $0.01 target.

---

## Key Design Decisions

**Why LangGraph over LangChain Expression Language (LCEL)?**
LangGraph gives you an explicit state machine with named nodes and conditional edges. When something breaks in production, you know exactly which node failed and why. LCEL chains are harder to debug and harder to extend with new routing logic.

**Why TypedDict for state?**
LangGraph's `AgentState` as a `TypedDict` makes state fields explicit and catches schema mismatches early. Avoid using plain dicts — silent field drops are a common source of hard-to-debug bugs.

**Why separate router and responder nodes?**
Mixing routing logic and generation in a single node creates a monolithic prompt that's hard to tune. Separating them lets you improve routing accuracy independently from response quality.

---

## Requirements

```
fastapi
uvicorn
langgraph
langchain
langchain-google-genai
qdrant-client
sentence-transformers
google-cloud-firestore
python-dotenv
pydantic
httpx
```

---

## Status

Active template — updated as new LangGraph patterns emerge in production use.

Based on architecture powering a deployed health AI assistant serving real users, with Qdrant vector search, multi-turn Firestore memory, and a tiered Gemini model setup.

---

## Author

**Fatima Tuz Zehra** — AI Engineer  
[LinkedIn](https://www.linkedin.com/in/fatima-tuz-zehra-678ba121a) · [GitHub](https://github.com/zehra99hassan) · zehra99hassan@gmail.com
