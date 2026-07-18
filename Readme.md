
# 🏢 Enterprise RAG System — Multi-Channel AI Knowledge Base with Zero-Trust Security

> A production-grade Retrieval-Augmented Generation system built with **n8n**, engineered to solve the three biggest problems companies face when deploying AI: **hallucination**, **data leakage**, and **silent failures**.

[![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?logo=n8n&logoColor=white)](https://n8n.io)
[![Supabase](https://img.shields.io/badge/Supabase-pgvector-3FCF8E?logo=supabase&logoColor=white)](https://supabase.com)
[![LlamaParse](https://img.shields.io/badge/LlamaParse-AI_Vision_Parsing-6C47FF)](https://cloud.llamaindex.ai)
[![Cohere](https://img.shields.io/badge/Cohere-Rerank_%2B_Embeddings-39594D?logo=cohere&logoColor=white)](https://cohere.com)
[![Groq](https://img.shields.io/badge/Groq-Whisper_%2B_Llama_3.3-F55036?logo=groq&logoColor=white)](https://groq.com)
[![Telegram](https://img.shields.io/badge/Telegram-Bot_API-26A5E4?logo=telegram&logoColor=white)](https://core.telegram.org/bots/api)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-Business_Cloud_API-25D366?logo=whatsapp&logoColor=white)](https://developers.facebook.com/docs/whatsapp)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎥 Proof of Work (Live Demos)

| Language | Type | Link |
|----------|------|------|
| 🇬 English | **Full Masterclass** — Architecture, decisions & live testing | [▶ Watch on YouTube](https://youtu.be/ZwlEWPxRcvY?si=wUkFHPLensH5IrCG) |
| 🇪🇬 Arabic  | **Walkthrough** — شرح تفصيلي للنظام بالكامل | [▶ شاهد على يوتيوب](https://youtu.be/t82bDGPduHw?si=PXy6Vbo8637Yjiqm) |

---

## 🎯 The Problem

Most enterprise AI deployments fail in three predictable ways:

| # | Problem | Business Impact |
|---|---------|-----------------|
| 1 | **Hallucination** — the LLM invents answers not present in company documents | Legal risk, loss of trust |
| 2 | **No access control** — any employee can query any document (salaries, HR, legal) | Data breach, compliance violation |
| 3 | **Silent failures** — when an AI provider goes down, users get no reply and admins get no alert | Operational blindness |

This system solves **all three** at the architecture level — not with prompt tricks.

---

## 🏗️ Architecture Overview

The system is composed of **three fully decoupled pipelines** with zero cross-pipeline coupling:

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. INGESTION PIPELINE                                              │
│     Google Drive (auto role-tag) ─┐                                 │
│                                    ├─► LlamaParse (AI Vision)       │
│     Webhook (API-key secured) ────┘     │                           │
│                                         ▼                           │
│     Duplicate detection + hard-delete of stale chunks               │
│                                         │                           │
│                                         ▼                           │
│     Cohere Multilingual Embeddings ─► Supabase (pgvector)           │
│                                         │                           │
│                                         ▼                           │
│                              Telegram admin notification            │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  2. RETRIEVAL & CHAT PIPELINE                                       │
│     Telegram / WhatsApp (text + voice)                              │
│              │                                                      │
│              ▼                                                      │
│     Groq Whisper STT (if voice)                                     │
│              │                                                      │
│              ▼                                                      │
│     Instant "typing" acknowledgment                                 │
│              │                                                      │
│              ▼                                                      │
│     RBAC check at SQL level (Zero-Trust)                            │
│              │                                                      │
│              ▼                                                      │
│     Hybrid Search (Vector + Full-Text) → RRF → top 20               │
│              │                                                      │
│              ▼                                                      │
│     Cohere Rerank → top 5 chunks                                    │
│              │                                                      │
│              ▼                                                      │
│     Custom Memory (Q/A pairs only, last 8 turns)                    │
│              │                                                      │
│              ▼                                                      │
│     LLM Chain (Primary + Fallback) → source-cited answer            │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  3. GLOBAL ERROR HANDLER                                            │
│     Auto-classify: AI Provider / Database / General                 │
│              │                                                      │
│     ┌────────┼─────────┐                                            │
│     ▼        ▼         ▼                                            │
│   Friendly   Admin     Persist to                                   │
│   user msg   Telegram  error_logs table                             │
│   (native    alert     (workflow, node, exec ID,                    │
│   channel)   (full      user ID, channel, time)                     │
│              forensic)                                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✨ Key Features

### 🔒 Zero-Trust Security
- **RBAC enforced at the SQL query level** — the LLM never even sees documents outside the user's role.
- Impossible to bypass via prompt injection, jailbreak, or any prompt-engineering technique.
- Per-document role tagging (auto-assigned from Google Drive folder, or via Webhook payload).

### 🧠 Anti-Hallucination Retrieval
- **LlamaParse AI-Vision** parsing — correctly extracts tables, images, and complex PDF layouts.
- **Hybrid Search** combining Vector Search + Full-Text Search via **Reciprocal Rank Fusion (RRF)**.
- **Cohere Rerank** distills top-20 candidates to the top-5 most relevant chunks — cutting cost, latency, and hallucination risk.
- Strict *"answer only from context, always cite source"* prompting policy.

### 💾 Custom Memory Architecture
- Stores **only** the user question and the AI answer — **never** the retrieved chunks.
- Prevents context-window bloat regardless of session length.
- User-triggered memory reset via natural-language command (Arabic & English).

### 🔄 Idempotent Ingestion
- Detects duplicate documents and **hard-deletes stale chunks** before re-ingestion.
- Adaptive retry/polling loop (up to 15-minute ceiling) for large-file parsing without timeouts.
- Soft-delete support from the admin dashboard.

### 🌐 Multi-Channel & Multi-Modal
- **Telegram** + **WhatsApp** from a single backend.
- **Text + Voice** on both channels (Groq Whisper STT, sub-second, Arabic & English).
- Instant acknowledgment messages for real-time perceived responsiveness.

### 🛡️ Production-Grade Error Handling
- Auto-classifies every failure into **AI Provider / Database / General**.
- Sends a friendly message to the user on their native channel.
- Fires a forensic alert to the admin Telegram group (workflow, failed node, execution ID, user ID, channel, timestamp, full error).
- Persists every incident to `error_logs` for analytics and reporting.

### 🔁 Dual-LLM Failover
- Primary LLM with automatic Fallback LLM — users never see a blank reply.

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| **Orchestration** | n8n (self-hosted) |
| **Vector DB** | Supabase (PostgreSQL + `pgvector`) |
| **Document Parsing** | LlamaParse (AI Vision) |
| **Embeddings** | Cohere Multilingual Embed v3 (Arabic + English) |
| **Reranking** | Cohere Rerank v3 |
| **Hybrid Fusion** | Reciprocal Rank Fusion (RRF) |
| **LLM** | Groq — Llama 3.3 70B (Primary + Fallback) |
| **Speech-to-Text** | Groq Whisper |
| **Channels** | Telegram Bot API, WhatsApp Business Cloud API |
| **Ingestion Sources** | Google Drive (folder-watch), secured Webhook |

---

## 🧪 Live Tests Demonstrated in the Videos

- ✅ Voice query (Arabic) → accurate answer with source citation.
- ✅ Text query on a document the user has **no role for** → politely refused (RBAC enforced).
- ✅ Same query after granting the role → answered with source.
- ✅ Memory reset command clears PostgreSQL history.
- ✅ Failure injection (broken Groq credentials) → Fallback LLM takes over.
- ✅ Failure injection with Fallback removed → user gets friendly message + admin gets forensic alert + row written to `error_logs`.

---



---

## 🧑‍💻 Author

**Yousef Abdelghaffar ElSherbiny** — AI Automation Engineer & n8n Specialist.

- 📺 YouTube: [@YousefAutomates](https://youtube.com/@YousefAutomates)
- 🐙 GitHub: [github.com/YousefAutomates](https://github.com/YousefAutomates)
- 📧 Email: yalsherbiny@gmail.com
- 📱 WhatsApp / Call: +20 101 010 4418

> Every automation skill I list is demonstrated live in publicly available video. No claimed skill lacks proof.

---

## 🇬 نبذة بالعربي

نظام **Enterprise RAG** مبني على **n8n** ومصمم لحل أكبر 3 مشاكل بتواجه الشركات لما بتستخدم الـ AI:
1. **الهلوسة** — الـ LLM بيألف إجابات مش موجودة في ملفات الشركة.
2. **تسريب البيانات** — أي موظف يقدر يسأل عن أي ملف (مرتبات، HR، قانوني).
3. **الأعطال الصامتة** — لما خدمة الـ AI تقع، المستخدم مياخدش رد والفريق ميعرفش.

النظام بيعالج المشاكل دي على مستوى الـ **Architecture** مش بالـ prompts:
- **RBAC على مستوى الـ SQL** (Zero-Trust) — الـ LLM مش بيشوف الملفات اللي مش من صلاحيته أصلاً.
- **Hybrid Search + RRF + Cohere Rerank** — دقة عالية وتكلفة أقل.
- **Custom Memory** — بيخزن السؤال والرد بس، مش الـ chunks.
- **Global Error Handler** — بيصنّف الخطأ، يرد على المستخدم، ينبّه الأدمن، ويسجّل كل حاجة في `error_logs`.

**شاهد الشرح الكامل:**
- 🇬🇧 [English Masterclass](https://youtu.be/ZwlEWPxRcvY?si=wUkFHPLensH5IrCG)
- 🇪🇬 [Arabic Walkthrough](https://youtu.be/t82bDGPduHw?si=PXy6Vbo8637Yjiqm)

---

## 📜 License

MIT © Yousef ElSherbiny
