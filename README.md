
# 🔍 Agentic Search for Context Engineering

A practical implementation of **Agentic Search** — the idea that ~80% of context engineering is deciding **what, when, and how to retrieve information** for LLMs.

Based on concepts from Leonie Monigatti's workshop at AI Engineer Europe 2026.

---

# 🎯 What This Project Demonstrates

| Problem | Solution |
|---|---|
| Traditional RAG always retrieves, even when unnecessary | Agent decides if retrieval is needed |
| Fixed RAG uses only one data source | Agent picks the right tool from a stack |
| Agents fail due to vague tool descriptions | Detailed descriptions with USE/DON'T USE guidance |
| LLMs write wrong SQL without schema knowledge | Agent Skills dynamically inject documentation |
| Shell tools are dangerous in production | Sandboxed file operations (list, read, grep only) |

---

# 🏗️ Architecture

```text
┌─────────────────────────────────────────────┐
│                 USER QUERY                  │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│         REACT AGENT (DeepSeek LLM)          │
│  Decides: Which tool? What parameters?      │
└──┬──────────┬──────────┬──────────┬─────────┘
   │          │          │          │
   ▼          ▼          ▼          ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────────┐
│Vector│  │ SQL  │  │Shell │  │  Agent   │
│Search│  │Query │  │Search│  │  Skills  │
└──┬───┘  └──┬───┘  └──┬───┘  └────┬─────┘
   │          │          │          │
   ▼          ▼          ▼          ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────────┐
│ Wiki │  │  DB  │  │Files │  │   Docs   │
│  +   │  │      │  │      │  │ (Schema, │
│ Docs │  │      │  │      │  │   API)   │
└──────┘  └──────┘  └──────┘  └──────────┘
````

---

# 🛠️ Tech Stack

| Component             | Technology                               | Purpose                          |
| --------------------- | ---------------------------------------- | -------------------------------- |
| **LLM**               | DeepSeek Chat (OpenAI-compatible API)    | Reasoning & tool selection       |
| **Embeddings**        | `sentence-transformers/all-MiniLM-L6-v2` | Free, local vector embeddings    |
| **Vector Store**      | ChromaDB                                 | Semantic similarity search       |
| **Agent Framework**   | LangChain + LangGraph                    | ReAct agent loop                 |
| **Database**          | SQLite                                   | Structured employee/project data |
| **Schema Validation** | Pydantic                                 | Tool input schemas               |

---

# 📦 5 Search Tools

## 1. `semantic_search` — Vector Search over Wiki

* Searches company wiki using semantic similarity
* ✅ Best for: policies, procedures, general knowledge
* ❌ Not for: structured data, aggregations

---

## 2. `query_database` — SQL Query Engine

* Executes SELECT queries on employee/project database
* ✅ Best for: employee info, salaries, project status
* ❌ Not for: policies, log files
* 🔒 Safety: Only SELECT queries allowed

---

## 3. `search_files` — Sandboxed File System Access

* Operations: list, read, grep
* ✅ Best for: log analysis, config lookups, meeting notes
* 🔒 Sandboxed: No write/delete, path traversal prevention

---

## 4. `load_db_schema` — Agent Skill (Dynamic)

* Loads database schema into LLM context on demand
* Reduces SQL errors from ~60% to ~10%
* Called before `query_database` when schema is unknown

---

## 5. `load_api_docs` — Agent Skill (Dynamic)

* Loads API documentation into context
* Saves tokens vs. putting all docs in system prompt

---

# 🚀 Quick Start

## 1. Open in Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](
https://colab.research.google.com/github/Yahia20/agentic-search-context-engineering/blob/main/agentic_search_context_engineering.ipynb
)

---

## 2. Set your API key

In Colab, go to **Secrets** (🔑 icon in sidebar) and add:

```env
DEEPSEEK_API_KEY=your_deepseek_api_key
```

Get one at:

[https://platform.deepseek.com/](https://platform.deepseek.com/)

Pricing starts at approximately **$0.27 / 1M tokens**.

---

## 3. Run all cells

The notebook is designed to run top-to-bottom with no modifications.

---

# 💰 Estimated Cost

Running all 6 demo queries costs approximately **$0.03** with DeepSeek.

---

# 🧪 Demo Queries

| # | Query                                                | Tools Used                                              | Type                |
| - | ---------------------------------------------------- | ------------------------------------------------------- | ------------------- |
| 1 | "What is the remote work policy?"                    | `semantic_search`                                       | Single tool         |
| 2 | "Who are the Engineering employees?"                 | `load_db_schema` → `query_database`                     | Skill + Tool        |
| 3 | "Are there errors in May 2nd logs?"                  | `search_files` (list + read)                            | Single tool         |
| 4 | "Jack Thomas wants a sabbatical..."                  | `load_db_schema` + `query_database` + `semantic_search` | Multi-tool          |
| 5 | "Database pool issue + who's the Security Engineer?" | `search_files` + `query_database`                       | Multi-source        |
| 6 | "What is 2 + 2?"                                     | None                                                    | No retrieval needed |
| 7 | Bad tool descriptions comparison                     | `search_bad` → `query_db_bad`                           | Failure demo        |

---

# 🧠 Key Concepts

## Context Engineering vs Prompt Engineering

* **Prompt Engineering** = How you phrase your question
* **Context Engineering** = What background information you provide

---

## From RAG → Agentic RAG → Agentic Search

```text
Traditional RAG:
Always retrieve → One source → Brittle ❌

Agentic RAG:
Agent decides → One source → Better ⚠️

Agentic Search:
Agent picks right tool → Multi-source → Robust ✅
```

---

## The 4 Failure Modes of Agentic Search

1. Not calling any tool (hallucination)
2. Calling the wrong tool
3. Generating wrong parameters
4. Calling too many tools

### Fix

Use detailed tool descriptions with clear:

* ✅ USE cases
* ❌ DON'T USE cases

---

## Agent Skills Pattern

Instead of putting all documentation in the system prompt (expensive), create a tool that loads it on demand.

Flow:

```text
Agent calls skill
    ↓
Gets documentation
    ↓
Uses other tools correctly
```

This reduces token usage while improving tool accuracy.

---

# 🔒 Production Considerations

* Use environment variables for API keys
* Add authentication & rate limiting
* Run shell tools in Docker containers
* Add LangSmith tracing for monitoring
* Use Redis for cached embeddings at scale
* Add retry logic and fallback tools

---

# 📚 References

* Workshop Video — AI Engineer Europe 2026
  [https://youtu.be/ynJyIKwjonM](https://youtu.be/ynJyIKwjonM)

* Workshop Repo by Leonie Monigatti
  [https://github.com/iamleonie/workshop-agentic-search](https://github.com/iamleonie/workshop-agentic-search)

* Leonie Monigatti on X
  [https://x.com/helloiamleonie](https://x.com/helloiamleonie)

* LangChain Documentation
  [https://python.langchain.com/](https://python.langchain.com/)

* LangGraph Documentation
  [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)

* ChromaDB Documentation
  [https://docs.trychroma.com/](https://docs.trychroma.com/)

---

# 📄 License

MIT License — feel free to use and adapt for your own projects.

```
```
