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
