<div align="center">

<!-- Language Switch Buttons -->
<a href="#arabic-version">
  <img src="https://img.shields.io/badge/🇸🇦_العربية-اقرأ_بالعربي-green?style=for-the-badge" alt="Arabic Version"/>
</a>
&nbsp;&nbsp;
<a href="#english-version">
  <img src="https://img.shields.io/badge/🇺🇸_English-Read_in_English-blue?style=for-the-badge" alt="English Version"/>
</a>

</div>

---

<div id="english-version">

<div align="center">

# 🏢 Enterprise RAG System
### Zero Hallucination · Role-Based Access · Production-Ready

<br/>

[![YouTube Masterclass](https://img.shields.io/badge/▶_Watch_Masterclass-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://youtu.be/t82bDGPduHw)
[![LinkedIn](https://img.shields.io/badge/Connect-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/YOUR-LINKEDIN)
[![Contact](https://img.shields.io/badge/Hire_Me-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://yousefautomates.pages.dev)

<br/>

> ⚠️ **This repository showcases the system architecture and capabilities.**
> Source code is available for enterprise clients and serious inquiries only.
> [Contact me directly →](https://yousefautomates.pages.dev)

</div>

---

## 🎬 Watch the Full Masterclass

<div align="center">

[![Enterprise RAG System Masterclass](https://img.youtube.com/vi/t82bDGPduHw/maxresdefault.jpg)](https://youtu.be/t82bDGPduHw)

**▶ [27-Minute Full Masterclass — Architecture + Live Demo](https://youtu.be/t82bDGPduHw)**

</div>

> 🌍 **Video is in Arabic** — targeting the 400M+ Arabic-speaking tech market
> where enterprise AI content is severely underserved.
>
> **To watch with English subtitles:**
> 1. Open the video on YouTube
> 2. Click `CC` (Subtitles) → `Auto-translate` → `English`
> 3. The architecture, code, and live demo are fully readable regardless of language.

<div align="center">

### 🇺🇸 English Version
![Coming Soon](https://img.shields.io/badge/English_Full_Version-Coming_Soon-orange?style=for-the-badge)

*Follow on [LinkedIn](https://www.linkedin.com/in/YOUR-LINKEDIN) to get notified when the English version drops.*

</div>

---

## 🔴 The Problem This System Solves

Most companies adopting AI face three critical failures in production:

| Failure | What Happens | Business Impact |
|---------|-------------|-----------------|
| **Hallucination** | LLM invents answers not in company documents | Legal liability, wrong decisions |
| **No Access Control** | All users access all documents | Data breach, compliance violation |
| **Silent Failures** | System crashes, users get no response | Customer trust destroyed |

**This system eliminates all three — by design.**

---

## ✅ System Capabilities

```
✅ Zero Hallucination     — Filtered at database level, not prompt level
✅ RBAC Enforcement       — Role-Based Access Control at SQL query time
✅ Zero Trust Security    — LLM never sees unauthorized data
✅ Hybrid Search          — Vector + Full-Text + RRF scoring combined
✅ Intelligent Reranking  — Cohere Rerank v3.5 selects top 5 from 20 chunks
✅ Voice Support          — Whisper-powered STT on Telegram + WhatsApp
✅ Custom Memory          — Stores Q&A only, never raw chunks
✅ Error Resilience       — Global handler with user notification + admin alert
✅ Multi-Channel          — Telegram and WhatsApp simultaneously
✅ Document Intelligence  — LlamaParse reads tables, images, complex PDFs
✅ Real-Time Monitoring   — Dashboard + error_logs table + Telegram alerts
✅ Soft Delete            — Document history preserved, chunks removed
✅ Race Condition Safety  — Database-level locking prevents duplication
```

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  ENTERPRISE RAG SYSTEM                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │         PIPELINE 1 — INGESTION                  │   │
│  │                                                 │   │
│  │  Google Drive ──┐                               │   │
│  │                 ├──► Role Assignment             │   │
│  │  Webhook API ───┘    (General / HR / Custom)    │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Trashed? ──► Soft Delete ──► Notify Admin      │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  LlamaParse (AI Vision Parser)                  │   │
│  │  Tables + Images + Complex PDFs                 │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Retry Loop (60 attempts × 15s)                 │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Chunk (400 tokens / 50 overlap)                │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Cohere Multilingual Embeddings                 │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Supabase (pgvector) ──► Status: Active         │   │
│  │  + Telegram Admin Notification                  │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │       PIPELINE 2 — RETRIEVAL & CHAT             │   │
│  │                                                 │   │
│  │  Telegram ──┐                                   │   │
│  │             ├──► Text / Voice ──► Groq Whisper  │   │
│  │  WhatsApp ──┘                                   │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Typing Indicator (UX Signal)                   │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  RBAC Lookup ──► user_role from PostgreSQL      │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Cohere Embeddings (Query)                      │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  hybrid_search_rbac()                           │   │
│  │  Vector Search + BM25 + RRF ──► Top 20 Chunks  │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Cohere Rerank v3.5 ──► Top 5 Chunks            │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Custom Memory (Last 8 Q&A pairs only)          │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Groq llama-3.3-70b + OpenAI Fallback           │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Reply with Source Citation                     │   │
│  │  ──► Telegram / WhatsApp                        │   │
│  │  ──► Log to chat_memory                         │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │       PIPELINE 3 — GLOBAL ERROR HANDLER         │   │
│  │                                                 │   │
│  │  Any Error ──► Lookup execution_context         │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  Classify: AI Provider / Database / Generic     │   │
│  │        │                                        │   │
│  │        ▼                                        │   │
│  │  ┌─────────────────────────────┐                │   │
│  │  │  User ◄── Friendly Message  │                │   │
│  │  │  Admin ◄── Full Error Alert │                │   │
│  │  │  DB ◄──── error_logs table  │                │   │
│  │  └─────────────────────────────┘                │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Orchestration** | n8n | Workflow automation engine |
| **Database** | PostgreSQL + pgvector | Vector + relational storage |
| **Vector Store** | Supabase | Cloud-hosted pgvector |
| **Search** | Hybrid BM25 + Vector | Dual-mode retrieval |
| **Ranking** | RRF Algorithm | Merges search results |
| **Reranking** | Cohere Rerank v3.5 | Selects top 5 from 20 |
| **Embeddings** | Cohere multilingual-v3.0 | Arabic + English support |
| **Primary LLM** | Groq llama-3.3-70b | Fast inference |
| **Fallback LLM** | OpenAI GPT | Reliability layer |
| **STT** | Groq Whisper-large-v3-turbo | Voice transcription |
| **Doc Parser** | LlamaParse | AI Vision parsing |
| **Channels** | Telegram + WhatsApp | Multi-channel interface |
| **Notifications** | Telegram Bot | Admin alerts |
| **Security** | RBAC + API Key + Zero Trust | Multi-layer protection |

---

## 🔐 Security Model

```
Layer 1: API Key Authentication
         Every webhook request validated before processing.

Layer 2: Role-Based Access Control (RBAC)
         user_role matched against document allowed_role
         at the SQL query level.

Layer 3: Zero Trust Architecture
         The LLM receives ONLY pre-filtered chunks.
         It is architecturally impossible for the LLM
         to access unauthorized data — even if it tries.

Layer 4: Database-Level Locking
         Race condition prevention via processing_started_at lock.
         Prevents duplicate ingestion under concurrent load.
```

---

## 📊 Live Demo Highlights

From the [27-minute Masterclass](https://youtu.be/t82bDGPduHw):

- ✅ **Voice query on WhatsApp** → Groq Whisper transcribes → RAG answers with source
- ✅ **RBAC Live Test** → User with General role asks HR-restricted question → Denied
- ✅ **Role Upgrade** → User granted HR role → Same question → Full answer with source
- ✅ **Error Simulation** → Groq API key corrupted → User gets friendly message, Admin gets full technical alert within seconds
- ✅ **Soft Delete** → Document removed from Supabase, history preserved in PostgreSQL
- ✅ **Deduplication** → Re-uploading same document clears old chunks before new ingestion

---

## 💼 Work With Me

This system was designed and built for enterprise deployment.

**I'm available for:**

| Engagement Type | Description |
|----------------|-------------|
| 🏢 **Full-Time Role** | AI Automation Engineer / ML Engineer |
| 🔧 **Consulting** | Enterprise RAG architecture and implementation |
| 🚀 **Freelance** | Custom AI systems for your organization |

**Source code and full technical documentation:**
Available to enterprise clients and serious inquiries.

<div align="center">

[![Contact Me](https://img.shields.io/badge/Get_In_Touch-Let's_Talk-blue?style=for-the-badge)](https://yousefautomates.pages.dev)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/YOUR-LINKEDIN)

</div>

---

</div>

<!-- ============================================================ -->
<!-- ARABIC VERSION SECTION -->
<!-- ============================================================ -->

<div id="arabic-version" dir="rtl">

---

<div align="center">

# 🏢 نظام Enterprise RAG
### بدون هلوسة · صلاحيات متكاملة · جاهز للإنتاج

<br/>

[![شاهد الشرح](https://img.shields.io/badge/▶_شاهد_الماستركلاس-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://youtu.be/t82bDGPduHw)
[![لينكدإن](https://img.shields.io/badge/تواصل_معي-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/YOUR-LINKEDIN)
[![الموقع](https://img.shields.io/badge/للتوظيف_والاستشارات-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://yousefautomates.pages.dev)

<br/>

> ⚠️ **هذا المستودع يعرض معمارية النظام وقدراته فقط.**
> الكود المصدري متاح للعملاء المؤسسيين والاستفسارات الجدية.
> [تواصل معي مباشرةً ←](https://yousefautomates.pages.dev)

</div>

---

## 🎬 شاهد الشرح الكامل

<div align="center">

[![Enterprise RAG System Masterclass](https://img.youtube.com/vi/t82bDGPduHw/maxresdefault.jpg)](https://youtu.be/t82bDGPduHw)

**▶ [ماستركلاس كاملة — 27 دقيقة — المعمارية + Demo حي](https://youtu.be/t82bDGPduHw)**

</div>

> 🌍 **لمشاهدة الفيديو بالترجمة الإنجليزية:**
> 1. افتح الفيديو على YouTube
> 2. اضغط `CC` (الترجمة) ← `Auto-translate` ← `English`

<div align="center">

### 🇺🇸 النسخة الإنجليزية
![قريباً](https://img.shields.io/badge/النسخة_الإنجليزية_الكاملة-قريباً-orange?style=for-the-badge)

*تابعني على [LinkedIn](https://www.linkedin.com/in/YOUR-LINKEDIN) لتصلك الإشعار عند إطلاقها.*

</div>

---

## 🔴 المشكلة التي يحلها هذا النظام

| المشكلة | ما يحدث | التأثير على الأعمال |
|---------|---------|-------------------|
| **الهلوسة** | الـ AI يخترع إجابات غير موجودة في الملفات | مشاكل قانونية وقرارات خاطئة |
| **لا صلاحيات** | كل المستخدمين يصلون لكل الملفات | اختراق البيانات وانتهاك الامتثال |
| **أعطال صامتة** | النظام يتوقف والمستخدم لا يعلم | تدمير ثقة العملاء |

**هذا النظام يُلغي المشاكل الثلاث — بالتصميم المعماري نفسه.**

---

## ✅ قدرات النظام

```
✅ Zero Hallucination     — فلترة على مستوى الداتابيز، ليس على مستوى الـ Prompt
✅ RBAC Enforcement       — تحكم في الصلاحيات على مستوى SQL
✅ Zero Trust Security    — الـ LLM لا يرى بيانات غير مصرح بها أبداً
✅ Hybrid Search          — Vector + Full-Text + RRF مجتمعة
✅ Intelligent Reranking  — Cohere Rerank v3.5 يختار أفضل 5 من 20 Chunk
✅ دعم الصوت             — Whisper على Telegram و WhatsApp
✅ Custom Memory          — يحفظ السؤال والإجابة فقط، لا الـ Chunks
✅ Error Resilience       — معالج أخطاء عالمي مع إشعار المستخدم والـ Admin
✅ Multi-Channel          — Telegram و WhatsApp في آنٍ واحد
✅ Document Intelligence  — LlamaParse يقرأ الجداول والصور والـ PDFs المعقدة
✅ Real-Time Monitoring   — Dashboard + error_logs + Telegram Alerts
✅ Soft Delete            — حفظ تاريخ الملفات مع حذف الـ Chunks
✅ Race Condition Safety  — Database-level Locking يمنع التكرار
```

---

## 🏗️ معمارية النظام (3 Pipelines)

**Pipeline 1 — الاستيعاب (Ingestion):**
Google Drive / Webhook ← تعيين الصلاحيات ← LlamaParse ← Chunking ← Cohere Embeddings ← Supabase

**Pipeline 2 — الاسترجاع والمحادثة (Retrieval & Chat):**
Telegram / WhatsApp ← Groq Whisper (للصوت) ← RBAC Lookup ← Hybrid Search ← Cohere Rerank ← LLM ← الرد بالمصدر

**Pipeline 3 — معالجة الأخطاء (Global Error Handler):**
أي خطأ ← تصنيف الخطأ ← رسالة ودية للمستخدم + تنبيه كامل للـ Admin + تسجيل في الداتابيز

---

## 🛠️ الـ Tech Stack

| الطبقة | التقنية | الغرض |
|-------|---------|--------|
| **Orchestration** | n8n | محرك الـ Workflows |
| **Database** | PostgreSQL + pgvector | تخزين المتجهات والعلاقات |
| **Vector Store** | Supabase | pgvector على السحابة |
| **Search** | Hybrid BM25 + Vector | استرجاع ثنائي الطريقة |
| **Ranking** | RRF Algorithm | دمج نتائج البحث |
| **Reranking** | Cohere Rerank v3.5 | أفضل 5 من 20 |
| **Embeddings** | Cohere multilingual-v3.0 | دعم العربية والإنجليزية |
| **LLM الرئيسي** | Groq llama-3.3-70b | استدلال سريع |
| **LLM البديل** | OpenAI GPT | طبقة موثوقية |
| **STT** | Groq Whisper-large-v3-turbo | تفريغ الصوت |
| **Parser** | LlamaParse | تحليل الملفات بالـ AI Vision |
| **القنوات** | Telegram + WhatsApp | واجهة متعددة القنوات |
| **الإشعارات** | Telegram Bot | تنبيهات الـ Admin |
| **الأمان** | RBAC + API Key + Zero Trust | حماية متعددة الطبقات |

---

## 📊 أبرز لحظات الـ Demo الحي

من [الماستركلاس (27 دقيقة)](https://youtu.be/t82bDGPduHw):

- ✅ **استفسار صوتي على WhatsApp** ← Groq Whisper يُفرّغ ← RAG يجيب بالمصدر
- ✅ **اختبار RBAC** ← مستخدم بصلاحية General يسأل عن ملف HR ← مرفوض
- ✅ **ترقية الصلاحية** ← المستخدم يأخذ صلاحية HR ← نفس السؤال ← إجابة كاملة بالمصدر
- ✅ **محاكاة الخطأ** ← Groq API معطوب ← المستخدم يتلقى رسالة ودية ← الـ Admin يتلقى تنبيهاً تقنياً كاملاً في نفس اللحظة
- ✅ **Soft Delete** ← الملف يُحذف من Supabase، التاريخ محفوظ في PostgreSQL
- ✅ **منع التكرار** ← إعادة رفع نفس الملف تحذف الـ Chunks القديمة قبل الجديدة

---

## 💼 تعامل معي

هذا النظام صُمّم وبُني للنشر المؤسسي الفعلي.

**متاح لـ:**

| نوع التعاون | التفاصيل |
|------------|---------|
| 🏢 **دوام كامل** | AI Automation Engineer / ML Engineer |
| 🔧 **استشارات** | معمارية وتنفيذ أنظمة Enterprise RAG |
| 🚀 **فريلانسر** | أنظمة AI مخصصة لشركتك |

<div align="center">

[![تواصل معي](https://img.shields.io/badge/تواصل_معي-للتوظيف_والاستشارات-blue?style=for-the-badge)](https://yousefautomates.pages.dev)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-تواصل-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/YOUR-LINKEDIN)

</div>

---

<div align="center">

*[العودة للنسخة الإنجليزية ↑](#english-version)*

</div>

</div>
