# Agent Pool Layer (`app/agents/`)

The **Agent Pool** contains specialized, stateless, single-responsibility LLM agents designed to execute as isolated nodes within a dynamically compiled [LangGraph](https://github.com/langchain-ai/langgraph) execution graph.

---

## 📌 Agent Roster & Functional Specialization

| Agent Node | Source File | Functional Responsibility | Primary Capabilities |
| :--- | :--- | :--- | :--- |
| **Planner** | [`planner.py`](planner.py) | Analyzes task complexity, extracts required capabilities, identifies external tools, and formats structured planning directives. | `planning`, `complexity_analysis`, `capability_identification` |
| **Researcher** | [`researcher.py`](researcher.py) | Conducts qualitative literature review, synthesizes facts, and structures multi-source knowledge. | `research`, `literature_review`, `synthesis` |
| **Coder** | [`coder.py`](coder.py) | Implements algorithms, produces production code snippets, and debugs logic problems. | `coding`, `algorithm_design`, `scripting` |
| **Critic** | [`critic.py`](critic.py) | Conducts strict verification, assesses solution quality ($0.0 - 1.0$), pinpoints syntax/logic errors, and triggers self-correction loops. | `verification`, `critique`, `self_correction` |
| **Tool Executor** | [`tool_executor.py`](tool_executor.py) | Dispatches native tool executions (web browsing, math arithmetic, arXiv querying, datetime fetching). | `tool_calling`, `web_search`, `calculation`, `retrieval` |
| **Finalizer** | [`finalizer.py`](finalizer.py) | Aggregates all intermediate agent outputs, cleans dialogue transcripts, and synthesizes publication-grade responses. | `final_synthesis`, `formatting`, `executive_summary` |
| **Registry** | [`registry.py`](registry.py) | Dynamic catalog managing agent metadata, capability bindings, and runtime instantiation. | `agent_discovery`, `schema_registry` |

---

## 🧩 Agent Node Contracts

Every agent operates strictly under a typed Pydantic contract:

### 1. `PlannerOutput`
```python
class PlannerOutput(BaseModel):
    task_understanding: str
    steps: List[str]
    required_capabilities: List[str]  # e.g., ["research", "coding", "web_search"]
    requires_research: bool
    requires_coding: bool
    requires_verification: bool
    requires_tools: bool
    tools_needed: List[str]            # e.g., ["web_search", "calculator"]
    estimated_complexity: str          # "simple", "moderate", "complex"
```

### 2. `CriticOutput` & Self-Correction Feedback Loop
```python
class CriticOutput(BaseModel):
    evaluation_score: float            # 0.0 to 1.0
    strengths: List[str]
    weaknesses: List[str]
    issues: List[str]                  # Specific bugs or deficits
    corrections: List[str]             # Prescriptive fixes
    verification_passed: bool          # True if score >= threshold (e.g., 0.75)
    missing_capabilities: List[str]    # Flags missing agents (triggers runtime recovery)
```

---

## ⚡ Self-Correction & Dynamic Recovery

1. **Threshold-Gated Rework Loop**: If `CriticOutput.evaluation_score < 0.80` or `verification_passed == False`, LangGraph cycles execution back to `Coder` or `Tool Executor` with exact issues, bounded by a maximum retry budget ($N \le 3$).
2. **Dynamic Missed-Agent Recovery**: If the initial adaptation pruned an agent (e.g., `tool_executor` on a disguised math prompt), the Critic flags `missing_capabilities`, prompting the orchestrator to inject the missing agent dynamically mid-execution.
