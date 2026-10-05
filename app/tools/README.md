# Tool Execution Layer (`app/tools/`)

The **Tool Execution Layer** provides safe, deterministic, external tool execution capabilities invoked by the dedicated `ToolExecutor` node in the dynamic graph.

---

## 🛠️ Integrated Tool Suite

| Tool Module | Key Function | External Dependency | Capabilities |
| :--- | :--- | :--- | :--- |
| [`web_search.py`](web_search.py) | `web_search()`, `duckduckgo_html_search()` | DuckDuckGo HTML / Tavily API | Real-time web browsing, news extraction, and search result summarization without requiring API keys. |
| [`arxiv_search.py`](arxiv_search.py) | `arxiv_search()` | `httpx`, ArXiv Atom API | Academic paper search, abstract extraction, citation harvesting, and author discovery. |
| [`calculator.py`](calculator.py) | `basic_calculator()` | Python AST (Safe Evaluator) | Exact arithmetic, mathematical calculations, percentages, power functions with zero LLM hallucination risk. |
| [`date_time.py`](date_time.py) | `get_current_date()`, `get_current_datetime()` | Python `datetime`, `zoneinfo` | Accurate real-world timestamps, current year anchoring, and temporal disambiguation. |

---

## 🔒 Safe Evaluation & Fallback Design

1. **AST Arithmetic Sandbox**: `calculator.py` uses AST tree validation to ensure no arbitrary code execution occurs when parsing mathematical expressions.
2. **Multi-Engine Web Search Fallback**: `web_search.py` attempts Tavily (if `TAVILY_API_KEY` is configured), immediately falling back to headless DuckDuckGo HTML parsing, ensuring continuous operation in air-gapped or keyless environments.
3. **Structured Tool Contracts**: Every tool returns structured dictionaries containing `status`, `result`, and `error` payloads for deterministic consumption by downstream LLM nodes.
