
<div align="center">

# 🤖 Enterprise RAG System — Multi-Channel AI Knowledge Base

### Production-grade Retrieval-Augmented Generation infrastructure built entirely in n8n

[![Arabic Description](https://img.shields.io/badge/🇪🇬_اقرأ_الوصف_بالعربية-اضغط_هنا-1F4E79?style=for-the-badge)](#arabic-description)

---

![Architecture](https://img.shields.io/badge/Architecture-3_Decoupled_Pipelines-1F4E79?style=flat-square)
![RAG](https://img.shields.io/badge/System-Enterprise_RAG-0A66C2?style=flat-square)
![RBAC](https://img.shields.io/badge/Security-RBAC_at_DB_Level-2E7D32?style=flat-square)
![Channels](https://img.shields.io/badge/Channels-Telegram_%2B_WhatsApp-6A1B9A?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production_Ready-00897B?style=flat-square)

</div>

---

## 🎬 Project Proof of Work

<div align="center">

| Resource | Link |
|----------|------|
| 🎥 **Full Walkthrough — English** | [![YouTube](https://img.shields.io/badge/YouTube-Coming_Soon-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](#) |
| 🎥 **Full Walkthrough — Arabic** | [![YouTube](https://img.shields.io/badge/يوتيوب-قريباً-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](#) |

> ⚙️ **Live demo video is in production — full architectural walkthrough with real data will be published soon.**

</div>

---

## 📋 Table of Contents

- [What This System Does](#what-this-system-does)
- [System Architecture](#system-architecture)
- [Pipeline 1 — Document Ingestion](#pipeline-1--document-ingestion)
- [Pipeline 2 — Retrieval & Chat](#pipeline-2--retrieval--chat)
- [Pipeline 3 — Global Error Handling](#pipeline-3--global-error-handling)
- [Tech Stack](#tech-stack)
- [Key Engineering Decisions](#key-engineering-decisions)
- [Arabic Description](#arabic-description)

---

## What This System Does

This is **not a chatbot demo**. This is a full production infrastructure that enables organisations to:

- **Ingest** any document (PDF, Drive files, or via API) into a searchable, permission-controlled knowledge base
- **Query** that knowledge base through natural language — via Telegram or WhatsApp — in text or voice
- **Enforce** role-based access control at the database level, not the prompt level
- **Survive** API failures gracefully — with user-friendly fallback messages and instant admin alerts
- **Scale** conversation history indefinitely without context bloat or performance degradation

The system moves organisations from "AI experiments" to **reliable, auditable AI automation**.

---

## System Architecture

```

┌─────────────────────────────────────────────────────────────┐
│                    ENTERPRISE RAG SYSTEM                    │
├──────────────────┬──────────────────┬───────────────────────┤
│   PIPELINE 1     │   PIPELINE 2     │   PIPELINE 3          │
│   Ingestion      │   Retrieval      │   Error Handling      │
│                  │   & Chat         │                       │
│ Webhook ──┐      │ Telegram ──┐     │ Any failure ──►       │
│           ├──►   │            ├──►  │ Classify ──►          │
│ G.Drive ──┘      │ WhatsApp ──┘     │ Notify user +         │
│                  │                  │ Alert admin +         │
│ Parse → Embed    │ Search → Rerank  │ Log to DB             │
│ → Store (RBAC)   │ → LLM → Reply    │                       │
└──────────────────┴──────────────────┴───────────────────────┘
│                                                             │
│              Supabase (pgvector + PostgreSQL)               │
└─────────────────────────────────────────────────────────────┘

```

---

## Pipeline 1 — Document Ingestion

**Goal:** Accept a document from any source, parse it intelligently, enforce permissions at the data level, and store it in a searchable vector format.

### Flow

```

[Webhook] ──────────────────────────────────────────────────────┐
                                                                 ▼
[Google Drive Trigger] ──► [Is File Trashed?] ──Yes──► [Delete Chunks] ──► [Notify]
                                    │
                                   No
                                    ▼
                          [Download File]
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │     Normalize Input           │
                    │  document_id / source_name /  │
                    │  source_type / allowed_role   │
                    └───────────────────────────────┘
                                    │
                                    ▼
                    [Delete Old Chunks — Postgres]   ◄── Data Integrity
                                    │
                                    ▼
                    [Upload to LlamaParse]
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │     Async Polling Loop        │
                    │  Check status every 15s       │
                    │  Max 60 retries (15 minutes)  │
                    └───────────────────────────────┘
                           │         │        │
                        SUCCESS    FAILED   TIMEOUT
                           │         │        │
                           ▼         ▼        ▼
                    [Get Markdown] [Notify] [Notify]
                           │
                           ▼
                    [Chunk + Metadata Injection]
                    allowed_role embedded per chunk   ◄── RBAC Foundation
                           │
                           ▼
                    [Cohere Multilingual Embeddings]
                           │
                           ▼
                    [Upsert → Supabase Vector Store]
                           │
                           ▼
                    [Update documents table → 'active']
                           │
                           ▼
                    [Telegram: Ingestion Success ✅]

```

### Key Design Decisions

| Decision | Reason |
|----------|--------|
| **Delete before re-ingest** | Prevents stale data hallucination when a document is updated |
| **Polling Loop outside the cycle** | Retry counter initialises once per document — prevents Race Conditions when parallel uploads occur |
| **LlamaParse over basic PDF parsers** | Preserves tables, columns, and complex layouts that plain text extraction destroys |
| **RBAC via Metadata Injection** | Security is baked into the data itself — not enforced by a prompt that can be bypassed |

---

## Pipeline 2 — Retrieval & Chat

**Goal:** Receive a question from any user on any channel, verify their permissions, retrieve the most relevant permitted content, and reply accurately — without memory bloat.

### Flow

```

[Telegram Trigger] ──┐
                      ├──► [Unsupported Media?] ──Yes──► [Polite Rejection]
[WhatsApp Trigger] ──┘           │
                                No
                                 │
                    ┌────────────┴────────────┐
                    │                         │
               [Is Voice?]              [Is Voice?]
                    │Yes                      │Yes
                    ▼                         ▼
            [Download Voice]         [Get Media URL]
                    │                         │
                    ▼                         ▼
            [Groq Whisper STT]      [Download + Groq Whisper STT]
                    │                         │
                    └────────────┬────────────┘
                                 ▼
                    [Normalize: text / session_id / channel]
                                 │
                                 ▼
                    [Is Reset Command?] ──Yes──► [Delete Memory] ──► [Confirm]
                                 │No
                                 ▼
                    [RBAC Lookup — Postgres]
                    user_role = actual role OR 'guest'
                                 │
                         ┌───────┴────────────────────────┐
                         ▼               ▼                ▼
                [Typing Indicator]  [Query Embedding]  [Fetch History]
                                         │                │
                                         ▼                │
                              [Hybrid Search — RBAC]      │
                              semantic + full-text        │
                              filtered at SQL level       │
                                         │                │
                                         ▼                │
                              [Cohere Rerank]             │
                              top 20 → top 5             │
                                         │                │
                                         └───────┬────────┘
                                                 ▼
                                    [Build System Message]
                                    context + history
                                    isolated in system prompt
                                                 │
                                                 ▼
                                    [Llama 3.3 70B — Groq]
                                                 │
                                                 ▼
                              ┌──────────────────┴───────────────────┐
                              ▼                                       ▼
                    [Send Reply — Telegram]              [Send Reply — WhatsApp]
                              │
                              └──────────────────┬───────────────────┘
                                                 ▼
                                    [Log to chat_memory]
                                    question + answer ONLY
                                    RAG chunks discarded   ◄── Anti-Context-Bloat

```

### Custom Memory Architecture — The Core Problem Solved

Standard Buffer Memory in RAG systems stores everything — questions, answers, **and** the retrieved chunks. After 10 questions, the context window carries 50+ stored chunks. This causes:

- ❌ Context Bloat — LLM performance degrades
- ❌ Increased token cost per query
- ❌ Hallucination from outdated context

**This system's solution:**

```

What gets stored in chat_memory:
✅ User question (short)
✅ Assistant answer (short)
❌ RAG chunks (discarded after use)

Result: Memory stays clean and bounded regardless of session length.

```

---

## Pipeline 3 — Global Error Handling

**Goal:** Ensure that any failure anywhere in the system results in a user-friendly message, an immediate admin alert, and a logged record — never a silent crash.

### Flow

```

[Any node failure in any pipeline]
                │
                ▼
    [Global Error Trigger — auto-fires]
                │
                ▼
    [Extract Context]
    workflow / node / error message / channel / user_id / timestamp
                │
                ▼
    [Classify Error — Regex on node name]
         │              │              │
    [AI Provider]  [Database]    [Unknown]
         │              │              │
         ▼              ▼              ▼
    [Build user-friendly fallback message per type]
                │
                ▼
    [Route by channel]
         │              │              │
    [Telegram]    [WhatsApp]    [No channel]
         │              │        (ingestion)
         ▼              ▼              │
    [Send fallback to user]            │
                │                      │
                └──────────┬───────────┘
                           ▼
            [Instant Admin Alert — Telegram]
            workflow + node + error + timestamp
                           │
                           ▼
            [Log to error_logs — Postgres]
            For dashboard analytics & SLA monitoring

```

### Error Classification

| Failure Source | User Message | Admin Alert |
|---------------|-------------|-------------|
| Groq / Cohere / LlamaParse | "AI servers are under load, please retry in a moment" | Full technical details |
| Postgres / Supabase | "Temporary database issue, our team is on it" | Full technical details |
| Unknown | "Temporary technical issue, we'll be back shortly" | Full technical details |
| Ingestion pipeline | *(no user to notify)* | Full technical details |

---

## Tech Stack

| Layer | Technology | Role |
|-------|-----------|------|
| **Orchestration** | n8n | All pipeline logic & workflow automation |
| **Document Parsing** | LlamaParse | Advanced PDF parsing (tables, layout, columns) |
| **Embeddings** | Cohere `embed-multilingual-v3.0` | Arabic + English semantic vectors |
| **Reranking** | Cohere `rerank-v3.5` | Precision context selection (top 20 → top 5) |
| **Vector Store** | Supabase pgvector | Hybrid search + RBAC filtering at SQL level |
| **Relational DB** | PostgreSQL (Supabase) | Users, documents, chat memory, error logs |
| **LLM Inference** | Groq + Llama 3.3 70B | Fast, cost-efficient answer generation |
| **Speech-to-Text** | Groq Whisper large-v3-turbo | Sub-second voice transcription |
| **Channels** | Telegram + WhatsApp Business API | Multi-platform user interface |
| **Notifications** | Telegram Bot API | Admin alerts + ingestion status |

---

## Key Engineering Decisions

### 1. RBAC at the Database Level

Security is enforced inside the SQL function `hybrid_search_rbac()`. The role filter runs before any data is returned. There is no prompt-level guard that can be bypassed.

### 2. Polling Loop with External Counter

The retry counter node (`04b`) lives **outside** the polling loop. This guarantees each document gets its own independent counter — eliminating Race Conditions when multiple documents are ingested simultaneously.

### 3. Hybrid Search

Combines **semantic search** (vector similarity via pgvector) with **full-text search** (PostgreSQL tsvector). Each retrieves what the other misses — the combination consistently outperforms either alone.

### 4. Decoupled Error Handler

Pipeline 3 is a completely separate workflow triggered by n8n's native error event. This means error handling logic is maintained in one place and applies to every pipeline automatically — no try/catch blocks scattered across nodes.

### 5. Graceful Degradation

When a failure occurs, the system degrades gracefully: the user receives a human-readable message, the admin receives a technical alert, and the error is logged. No silent crashes. No technical error screens exposed to end users.

---

## Database Schema Overview

```sql
-- Documents registry
documents (document_id, source_name, source_type, status, updated_at)

-- Vector chunks with RBAC metadata
document_chunks (id, content, embedding vector(1024), 
                 document_id, source_name, allowed_role)

-- User roles for RBAC
users (id, chat_id, phone_number, role)

-- Conversation history (clean — no RAG chunks stored)
chat_memory (id, session_id, channel, role, content, created_at)

-- Error telemetry
error_logs (id, workflow_name, node_name, error_message, 
            session_id, created_at)
```

---

## Author

**Yousef Abdelghaffar ElSherbiny**
AI Automation Engineer | n8n Specialist | Enterprise RAG Systems Builder

[![GitHub](https://img.shields.io/badge/GitHub-YousefAutomates-181717?style=flat-square&logo=github)](https://github.com/YousefAutomates)
[![YouTube](https://img.shields.io/badge/YouTube-@YousefAutomates-FF0000?style=flat-square&logo=youtube)](https://youtube.com/@YousefAutomates)

---

---

<div dir="rtl">

<a name="arabic-description"></a>

## 🇪🇬 وصف المشروع — Enterprise RAG System

### 🎬 إثبات المشروع

| المورد | الرابط |
|--------|--------|
| 🎥 **شرح المشروع كامل بالعربي** | [![يوتيوب](https://img.shields.io/badge/يوتيوب-قريباً-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](#) |
| ⚙️ **عرض النظام مباشر** | [![لايف](https://img.shields.io/badge/عرض_مباشر-قريباً-1F4E79?style=for-the-badge)](#) |

> 📹 **فيديو الشرح والعرض المباشر قيد التصوير — سيُنشر قريباً بشرح معماري كامل مع بيانات حقيقية.**

---

### ما الذي يفعله هذا النظام؟

هذا النظام **ليس مجرد شات بوت** — هو بنية تحتية إنتاجية متكاملة تمكّن المؤسسات من:

- **فهرسة** أي مستند (PDF، Google Drive، أو عبر API) في قاعدة معرفة قابلة للبحث ومحمية بصلاحيات دقيقة
- **الاستعلام** عن قاعدة المعرفة بلغة طبيعية — عبر Telegram أو WhatsApp — بالنص أو الصوت
- **تطبيق** نظام صلاحيات متدرج (RBAC) على مستوى قاعدة البيانات مباشرة، لا على مستوى الـ Prompt
- **الصمود** أمام أعطال الـ API بشكل رشيق — مع رسائل اعتذار مناسبة للمستخدم وتنبيهات فورية للإدارة
- **التوسع** في سجلات المحادثة إلى ما لا نهاية دون تضخم في السياق أو تراجع في الأداء

---

### معمارية النظام — 3 مسارات مستقلة

**المسار الأول — الاستيعاب والفهرسة:**
استقبال الملفات من Webhook أو Google Drive ← تحليل ذكي عبر LlamaParse (يحافظ على الجداول والتخطيط) ← تقطيع مع حقن الصلاحيات في كل قطعة ← تحويل لـ Embeddings عبر Cohere متعدد اللغات ← تخزين في Supabase pgvector.

**المسار الثاني — الاسترجاع والدردشة:**
استقبال من Telegram وWhatsApp ← دعم الرسائل الصوتية عبر Groq Whisper ← التحقق من هوية المستخدم وصلاحياته ← Hybrid Search مع فلترة RBAC على مستوى SQL ← Cohere Rerank (أفضل 5 من أصل 20) ← Llama 3.3 70B على Groq للإجابة ← ذاكرة مخصصة لا تحفظ القطع الضخمة، فقط السؤال والإجابة.

**المسار الثالث — معالجة الأخطاء المركزية:**
التقاط تلقائي لأي خطأ في أي مسار ← تصنيف الخطأ ← رسالة لطيفة للمستخدم على قناته الصحيحة ← تنبيه فوري للإدارة على Telegram ← تسجيل في قاعدة البيانات للتحليل.

---

### الحلول الهندسية الجوهرية

**١. مشكلة تضخم الذاكرة (Context Bloat) — محلولة:**
أنظمة الـ RAG التقليدية تحفظ القطع المسترجعة في الذاكرة. بعد 10 أسئلة = 50+ قطعة محفوظة = سياق ضخم = أداء متردٍّ وتكلفة مرتفعة. هنا: القطع تُستخدم وتُتجاهل. الذاكرة تحفظ السؤال والإجابة فقط.

**٢. الأمان على مستوى قاعدة البيانات:**
الصلاحيات محقونة في بيانات كل قطعة نص ومفلترة بدالة SQL. لا يمكن لأي سؤال — مهما كان — استخراج محتوى خارج صلاحية المستخدم.

**٣. حلقة الانتظار الذكية:**
تحليل الملفات الكبيرة يأخذ وقتاً. النظام يسأل كل 15 ثانية (حتى 60 مرة = 15 دقيقة) بدلاً من الوقوع بـ Timeout.

**٤. الانحدار الرشيق (Graceful Degradation):**
عند أي عطل، المستخدم يرى رسالة إنسانية مناسبة — لا شاشة خطأ تقنية. الإدارة تتلقى التفاصيل التقنية الكاملة فوراً.

---

### التقنيات المستخدمة

| الطبقة | التقنية |
|--------|---------|
| التنسيق | n8n |
| تحليل المستندات | LlamaParse |
| التضمينات | Cohere Multilingual |
| إعادة الترتيب | Cohere Rerank |
| قاعدة البيانات المتجهية | Supabase pgvector |
| نموذج اللغة | Llama 3.3 70B / Groq |
| تحويل الصوت لنص | Groq Whisper |
| القنوات | Telegram + WhatsApp Business |

---

### 

**يوسف عبد الغفار الشربيني**
مهندس أتمتة وذكاء اصطناعي | متخصص n8n | بناء أنظمة RAG المؤسسية

[![GitHub](https://img.shields.io/badge/GitHub-YousefAutomates-181717?style=flat-square&logo=github)](https://github.com/YousefAutomates)
[![YouTube](https://img.shields.io/badge/يوتيوب-@YousefAutomates-FF0000?style=flat-square&logo=youtube)](https://youtube.com/@YousefAutomates)

</div>
