# Conversational Memory & Persistence Layer (`app/memory/`)

The **Memory & Persistence Layer** manages multi-turn conversation threads, LangGraph checkpointers, message serialization, and persistent storage using MongoDB Atlas with an automatic offline local fallback.

---

## 📌 Capabilities

1. **LangGraph Checkpoint Integration**: Persists conversation states and node execution checkpoints per `thread_id`.
2. **Hybrid Storage Architecture**:
   - **MongoDB Atlas**: High-availability cloud persistence for threads, message history, run metrics, Q-tables, and generated graph PNG visualizations.
   - **Graceful Local Fallback**: When MongoDB is offline or unconfigured, gracefully switches to local in-memory checkpointing and `data/q_table.json` without failing runtime operations.
3. **Dialogue Filtering & Serialization**: Cleans internal agent reasoning scratchpads and extracts pure human-assistant conversational dialogue for prompt injection and UI consumption.
4. **User & Thread Isolation**: Strict scoping of messages, execution histories, and metrics by `user_id` and `thread_id`.

---

## 📂 Module Files

| File | Core Classes / Functions | Description |
| :--- | :--- | :--- |
| [`store.py`](store.py) | `ThreadMessageStore`, `is_dialogue_message()`, `serialize_message()` | Multi-turn message storage, LangGraph checkpointer management, and dialogue serialization. |
| [`mongo_client.py`](mongo_client.py) | `get_mongo_client()`, `get_mongo_db()`, `check_mongo_network_error()` | Resilient MongoDB Atlas client lifecycle management and error recovery. |

---

## 🗄️ Database Collections Schema (MongoDB Atlas)

- `threads`: Stores thread metadata, snippet previews, message count, and user associations.
- `messages`: Serialized chat dialogue history scoped to `(thread_id, user_id)`.
- `runs`: Full execution traces, token telemetry, latency, and RL reward breakdowns.
- `q_table`: Persistent state-action Q-values for the RL policy.
- `graphs`: Base64 encoded PNG rendered graphs of the latest topology per thread.
