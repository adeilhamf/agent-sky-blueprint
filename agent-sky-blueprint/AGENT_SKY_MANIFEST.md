# AGENT SKY MANIFEST
Version: 0.2 (Engineering Requirement Edition)

---

# PART 1 — FOUNDATION REQUIREMENTS

## 1. Core Identity

REQ-0001  
Agent Sky adalah platform AI self-hosted yang berjalan di komputer pribadi menggunakan Docker Compose.

REQ-0002  
Agent Sky bukan SaaS dan tidak dirancang untuk multi-tenant publik.

REQ-0003  
Seluruh sistem harus bisa berjalan secara offline dengan AI lokal (Ollama) sebagai fallback.

REQ-0004  
Sistem harus AI-provider agnostic (tidak boleh bergantung pada satu provider AI).

REQ-0005  
Semua AI request harus melalui AI Provider Layer.

---

## 2. Architecture Rules

REQ-0010  
Sistem harus menggunakan modular architecture berbasis service.

REQ-0011  
Tidak boleh ada monolithic logic yang mencampur UI, API, dan business logic.

REQ-0012  
Semua service harus bisa dijalankan secara independen via Docker.

REQ-0013  
Semua komunikasi antar service harus melalui API atau event system.

REQ-0014  
Tidak boleh ada direct database access dari frontend.

REQ-0015  
Semua dependency harus explicit, tidak boleh hidden global dependency.

---

## 3. Backend Rules

REQ-0020  
Backend wajib menggunakan FastAPI.

REQ-0021  
Semua endpoint harus versioned (/v1/).

REQ-0022  
Semua response harus memiliki format standar:

{
  "success": true,
  "data": {},
  "error": null,
  "meta": {}
}

REQ-0023  
Semua error harus memiliki error code yang konsisten.

REQ-0024  
Tidak boleh ada business logic di router layer.

REQ-0025  
Business logic harus berada di service layer.

---

## 4. Frontend Rules

REQ-0030  
Frontend wajib menggunakan Next.js + TypeScript.

REQ-0031  
Frontend tidak boleh mengakses database langsung.

REQ-0032  
Frontend hanya boleh berkomunikasi dengan Backend API.

REQ-0033  
UI harus modular dan reusable component-based.

REQ-0034  
Tidak boleh ada business logic kompleks di frontend.

---

## 5. Data & Database Rules

REQ-0040  
Database utama adalah PostgreSQL.

REQ-0041  
Redis digunakan hanya untuk cache, queue, dan session.

REQ-0042  
Semua migration harus menggunakan Alembic.

REQ-0043  
Tidak boleh ada schema change tanpa migration file.

REQ-0044  
Semua tabel harus memiliki:
- id (UUID)
- created_at
- updated_at

---

## 6. AI System Rules

REQ-0050  
Semua AI request harus melalui AI Provider Layer.

REQ-0051  
Provider harus dapat diganti tanpa mengubah business logic.

REQ-0052  
Provider awal adalah 9Router.

REQ-0053  
Provider fallback adalah Ollama (local model).

REQ-0054  
Tidak boleh ada hardcoded prompt di business logic.

REQ-0055  
Semua prompt harus disimpan di prompt registry.

---

## 7. Agent System Rules

REQ-0060  
Semua agent harus berjalan sebagai isolated module.

REQ-0061  
Agent tidak boleh mengakses filesystem secara bebas tanpa Tool SDK.

REQ-0062  
Agent hanya boleh melakukan aksi melalui Tools.

REQ-0063  
Semua agent harus memiliki lifecycle standar:
- Goal
- Plan
- Execute
- Review
- Done

REQ-0064  
Tidak boleh ada agent yang langsung deploy production.

---

## 8. Tool System Rules

REQ-0070  
Semua external actions harus melalui Tool SDK.

REQ-0071  
Tool harus bersifat stateless.

REQ-0072  
Tool harus bisa diuji secara unit test.

REQ-0073  
Tool tidak boleh memiliki dependency ke agent.

---

## 9. Workspace Rules

REQ-0080  
Semua pekerjaan harus berada dalam Workspace context.

REQ-0081  
Workspace harus terisolasi per project.

REQ-0082  
Agent tidak boleh mengakses workspace lain tanpa izin.

REQ-0083  
Workspace harus memiliki status lifecycle:
- active
- archived
- paused

---

## 10. Security Rules

REQ-0090  
Tidak boleh ada secret di dalam repository.

REQ-0091  
Semua secret harus menggunakan environment variables.

REQ-0092  
Tidak boleh ada API key hardcoded.

REQ-0093  
Semua request harus bisa di-log untuk audit.

---

## END OF PART 1
# PART 2 — ARCHITECTURE & SYSTEM DESIGN

---

# 11. High Level Architecture

REQ-0100  
Agent Sky harus memiliki layered architecture yang jelas:

Browser / User
→ Frontend (Next.js)
→ Backend API (FastAPI)
→ Core Engine
→ Agent Runtime
→ Tools Layer
→ External Systems

---

REQ-0101  
Tidak boleh ada direct communication dari Frontend ke Core Engine atau Agents.

---

REQ-0102  
Semua komunikasi harus melalui Backend API sebagai gateway.

---

## Architecture Diagram (Logical)

```text
User
 ↓
Frontend (Next.js)
 ↓
Backend API (FastAPI)
 ↓
Core Engine
 ├── Agent Manager
 ├── Workspace Manager
 ├── AI Provider Layer
 ├── Job Queue (Redis)
 ├── Memory System
 └── Tool SDK
 ↓
Agents
 ↓
Tools
 ↓
External Systems
```

---

# 12. Core Engine Design

REQ-0110  
Core Engine adalah pusat kontrol semua proses Agent Sky.

REQ-0111  
Core Engine tidak boleh tergantung pada UI atau Frontend.

REQ-0112  
Core Engine harus stateless sebanyak mungkin.

REQ-0113  
State harus disimpan di Database atau Redis.

---

## Core Modules

REQ-0114  
Core Engine terdiri dari modul berikut:

- Agent Manager
- Workspace Manager
- AI Provider Layer
- Memory System
- Job Queue Manager
- Tool Registry
- Event Bus

---

# 13. Event-Driven System

REQ-0120  
Agent Sky harus menggunakan event-driven architecture untuk semua workflow agent.

---

REQ-0121  
Semua aksi penting harus menghasilkan event.

Contoh event:

- agent.created
- task.assigned
- task.completed
- job.failed
- workspace.updated

---

REQ-0122  
Event harus dapat disimpan untuk audit trail.

---

REQ-0123  
Event harus dapat di-replay untuk debugging.

---

# 14. Agent Runtime System

REQ-0130  
Semua agent harus berjalan di dalam Agent Runtime Layer.

---

REQ-0131  
Agent tidak boleh berjalan langsung di backend API layer.

---

REQ-0132  
Agent lifecycle wajib mengikuti struktur ini:

1. Receive Goal
2. Create Plan
3. Break into Tasks
4. Request Approval (if needed)
5. Execute Tasks
6. Validate Result
7. Generate Report
8. Finish

---

REQ-0133  
Agent tidak boleh melewati langkah Planning.

---

REQ-0134  
Agent harus menghasilkan output structured (JSON or schema-based).

---

# 15. AI Provider Layer

REQ-0140  
Semua request ke LLM harus melalui AI Provider Layer.

---

REQ-0141  
AI Provider Layer harus mendukung multiple providers:

- 9Router (primary)
- OpenAI
- Anthropic
- Gemini
- Ollama (local fallback)

---

REQ-0142  
Switching provider tidak boleh mengubah business logic.

---

REQ-0143  
Prompt harus dipisahkan dari code logic (Prompt Registry).

---

REQ-0144  
AI Provider Layer harus memiliki retry & fallback system.

---

# 16. Job Queue System

REQ-0150  
Semua task asynchronous harus menggunakan Redis-based job queue.

---

REQ-0151  
Job harus memiliki status:

- pending
- running
- completed
- failed
- retrying

---

REQ-0152  
Job harus idempotent (tidak boleh double execution).

---

REQ-0153  
Job harus bisa di-retry otomatis dengan backoff strategy.

---

# 17. Workspace System

REQ-0160  
Workspace adalah unit kerja utama Agent Sky.

---

REQ-0161  
Setiap project harus memiliki workspace terisolasi.

---

REQ-0162  
Workspace harus menyimpan:

- files
- logs
- agent history
- tasks
- memory context

---

REQ-0163  
Workspace tidak boleh diakses lintas project tanpa permission.

---

# 18. Memory System

REQ-0170  
Memory adalah sistem penyimpanan pengetahuan jangka panjang Agent Sky.

---

REQ-0171  
Memory harus dapat diakses oleh semua agent.

---

REQ-0172  
Memory harus menyimpan:

- decisions
- architecture
- user preferences
- project knowledge
- past executions

---

REQ-0173  
Memory harus versioned (tidak overwrite langsung).

---

# 19. Tool System

REQ-0180  
Semua aksi external harus melalui Tool SDK.

---

REQ-0181  
Tool harus memiliki interface standar:

- input schema
- output schema
- error schema

---

REQ-0182  
Tool tidak boleh bergantung pada agent logic.

---

REQ-0183  
Tool harus bisa diuji secara independen.

---

# 20. Data Flow Standard

REQ-0190  
Semua data flow harus mengikuti pola berikut:

User Input
→ API Request
→ Core Engine
→ Agent Runtime
→ Tools
→ Result
→ Memory Update
→ Response

---

# END OF PART 2
# PART 3 — ENGINEERING STANDARDS & CODE RULES

---

# 21. Backend Structure (FastAPI)

REQ-0200  
Backend wajib menggunakan FastAPI dengan struktur modular berbasis domain.

---

REQ-0201  
Struktur folder backend harus:

```
backend/
├── app/
│   ├── api/
│   │   ├── v1/
│   │   ├── routes/
│   │   └── middleware/
│   │
│   ├── core/
│   │   ├── config/
│   │   ├── security/
│   │   └── logging/
│   │
│   ├── modules/
│   │   ├── agents/
│   │   ├── workspace/
│   │   ├── ai_provider/
│   │   ├── memory/
│   │   └── jobs/
│   │
│   ├── db/
│   │   ├── models/
│   │   ├── migrations/
│   │   └── session/
│   │
│   ├── services/
│   ├── schemas/
│   └── main.py
```

---

REQ-0202  
Tidak boleh ada business logic di `api/routes`.

---

REQ-0203  
Semua business logic harus berada di `services/`.

---

REQ-0204  
Semua database query hanya boleh diakses melalui `db/session` atau repository layer.

---

# 22. Frontend Structure (Next.js)

REQ-0210  
Frontend wajib menggunakan Next.js App Router.

---

REQ-0211  
Struktur folder frontend:

```
frontend/
├── app/
│   ├── dashboard/
│   ├── workspace/
│   ├── agents/
│   └── layout.tsx
│
├── components/
├── hooks/
├── lib/
├── services/
├── types/
└── styles/
```

---

REQ-0212  
Frontend tidak boleh mengandung business logic kompleks.

---

REQ-0213  
Semua API call harus melalui `services/` layer.

---

# 23. API Standards

REQ-0220  
Semua API harus menggunakan prefix `/api/v1`.

---

REQ-0221  
Semua response harus mengikuti format standar:

```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {}
}
```

---

REQ-0222  
Semua error harus memiliki:

- error_code
- message
- details (optional)

---

REQ-0223  
Tidak boleh ada response format yang berbeda di seluruh system.

---

# 24. Logging System

REQ-0230  
Semua service wajib memiliki structured logging.

---

REQ-0231  
Log format wajib JSON.

---

REQ-0232  
Log harus mencakup:

- timestamp
- service name
- request_id
- level
- message

---

REQ-0233  
Tidak boleh ada print() untuk debugging di production code.

---

# 25. Error Handling Standard

REQ-0240  
Semua error harus ditangani secara explicit.

---

REQ-0241  
Tidak boleh ada silent failure.

---

REQ-0242  
Semua error harus memiliki:

- error_code
- human readable message
- stack trace (dev only)

---

REQ-0243  
Core Engine harus dapat membedakan:

- system error
- agent error
- tool error
- validation error

---

# 26. Configuration System

REQ-0250  
Semua konfigurasi harus berasal dari environment variables.

---

REQ-0251  
Tidak boleh ada hardcoded configuration.

---

REQ-0252  
Semua config harus divalidasi saat startup.

---

REQ-0253  
Jika config invalid, system harus fail fast.

---

# 27. Dependency Rules

REQ-0260  
Dependency harus explicit.

---

REQ-0261  
Tidak boleh ada hidden global state.

---

REQ-0262  
Dependency Injection harus digunakan untuk service layer.

---

REQ-0263  
Circular dependency tidak diperbolehkan.

---

# 28. Testing Strategy

REQ-0270  
Semua module harus memiliki unit test.

---

REQ-0271  
Backend menggunakan pytest.

---

REQ-0272  
Frontend menggunakan Vitest.

---

REQ-0273  
Setiap Sprint harus memiliki test coverage minimum 70%.

---

REQ-0274  
Tidak boleh merge jika test gagal.

---

# 29. Git & Versioning Rules

REQ-0280  
Branch utama:

- main (production)
- develop (integration)
- feature/*

---

REQ-0281  
Tidak boleh commit langsung ke main.

---

REQ-0282  
Setiap Sprint harus memiliki tag version.

---

REQ-0283  
Commit message harus format:

type(scope): message

Contoh:
feat(agent): add coding agent planner

---

# 30. Code Quality Rules

REQ-0290  
Semua code harus mengikuti:

Python:
- PEP8
- Ruff
- Black

TypeScript:
- ESLint
- Prettier
- Strict Mode

---

REQ-0291  
Tidak boleh ada unused code.

REQ-0292  
Tidak boleh ada duplicated logic tanpa abstraction.

REQ-0293  
Semua function harus single responsibility.

---

# END OF PART 3

# PART 4 — AI SYSTEM DESIGN & COGNITIVE ARCHITECTURE

---

# 31. AI Core Principle

REQ-0300  
Semua AI di Agent Sky harus mengikuti struktur berpikir yang terstandarisasi.

---

REQ-0301  
AI tidak boleh langsung mengeksekusi tugas tanpa planning.

---

REQ-0302  
Semua AI output harus melalui structured reasoning pipeline.

---

REQ-0303  
AI tidak boleh menyimpan state sendiri di luar Memory System.

---

# 32. Cognitive Pipeline (Wajib)

REQ-0310  
Semua agent wajib mengikuti pipeline berikut:

```
Goal
↓
Context Builder
↓
Planner
↓
Task Decomposition
↓
Approval Gate (optional)
↓
Executor
↓
Validator
↓
Reviewer
↓
Memory Writer
↓
Response
```

---

REQ-0311  
AI tidak boleh melewati stage Planner.

---

REQ-0312  
AI tidak boleh langsung execute task kompleks tanpa breakdown.

---

# 33. Context Builder System

REQ-0320  
Setiap AI request harus melewati Context Builder.

---

REQ-0321  
Context Builder harus menggabungkan:

- User input
- Workspace state
- Memory system
- Previous tasks
- Agent history

---

REQ-0322  
Context Builder harus melakukan filtering agar context tidak overload.

---

REQ-0323  
Context Builder harus mendukung token optimization strategy.

---

# 34. Planner System

REQ-0330  
Planner bertanggung jawab mengubah goal menjadi structured tasks.

---

REQ-0331  
Planner harus menghasilkan output dalam format:

```json
{
  "goal": "",
  "tasks": [
    {
      "id": "task_1",
      "description": "",
      "dependencies": [],
      "estimated_complexity": "low | medium | high"
    }
  ]
}
```

---

REQ-0332  
Planner tidak boleh mengeksekusi task.

---

REQ-0333  
Planner harus selalu deterministic untuk input yang sama.

---

# 35. Executor System

REQ-0340  
Executor hanya menjalankan task yang sudah dihasilkan Planner.

---

REQ-0341  
Executor tidak boleh membuat task baru tanpa izin Planner.

---

REQ-0342  
Executor hanya boleh menggunakan Tool SDK untuk aksi eksternal.

---

REQ-0343  
Executor harus mengembalikan structured result per task.

---

# 36. Reviewer System

REQ-0350  
Reviewer bertugas mengevaluasi hasil Executor.

---

REQ-0351  
Reviewer harus memvalidasi:

- correctness
- completeness
- consistency
- safety

---

REQ-0352  
Reviewer dapat meminta re-execution jika hasil tidak valid.

---

REQ-0353  
Reviewer tidak boleh mengubah hasil langsung.

---

# 37. Memory Injection System

REQ-0360  
Memory harus di-inject ke AI context sebelum Planner berjalan.

---

REQ-0361  
Memory harus dipilih secara relevan (semantic filtering).

---

REQ-0362  
Tidak boleh semua memory dimasukkan sekaligus.

---

REQ-0363  
Memory harus versioned dan traceable.

---

REQ-0364  
AI harus bisa menulis ke memory hanya melalui Memory Writer stage.

---

# 38. Tool Calling System

REQ-0370  
AI tidak boleh mengakses system langsung tanpa Tool SDK.

---

REQ-0371  
Semua tool call harus melalui structured schema.

---

REQ-0372  
Tool output harus validatable.

---

REQ-0373  
Tool failure harus dikembalikan sebagai structured error.

---

REQ-0374  
AI harus retry tool call jika gagal (max 3 kali).

---

# 39. Multi-Agent Communication

REQ-0380  
Agent Sky harus mendukung komunikasi antar agent.

---

REQ-0381  
Agent tidak boleh saling memanggil langsung.

---

REQ-0382  
Semua komunikasi harus melalui Event Bus.

Contoh event:

- agent.request
- agent.response
- agent.task.transfer

---

REQ-0383  
Tidak boleh ada circular agent dependency.

---

# 40. AI Provider Abstraction (9Router Layer)

REQ-0390  
Semua AI request harus melalui AI Provider Abstraction Layer.

---

REQ-0391  
9Router adalah default router untuk LLM selection.

---

REQ-0392  
Provider harus bisa diganti tanpa mengubah logic agent.

---

REQ-0393  
Provider wajib mendukung:

- OpenAI
- Anthropic
- Gemini
- Ollama
- Custom endpoint

---

REQ-0394  
Provider harus mendukung fallback chain otomatis.

---

REQ-0395  
Provider harus mencatat semua request untuk audit log.

---

# END OF PART 4
# PART 5 — CODING AGENT DESIGN

---

# 41. Coding Agent Definition

REQ-0400  
Coding Agent adalah agent utama yang bertugas memodifikasi, membuat, dan memperbaiki kode dalam repository Agent Sky.

---

REQ-0401  
Coding Agent tidak boleh melakukan deployment ke production environment tanpa approval manusia.

---

REQ-0402  
Coding Agent hanya boleh bekerja dalam workspace yang ditentukan.

---

REQ-0403  
Coding Agent harus berbasis task-driven execution, bukan free-form coding.

---

# 42. Coding Agent Lifecycle

REQ-0410  
Coding Agent wajib mengikuti lifecycle berikut:

```
Goal
↓
Analysis
↓
Planning
↓
Task Breakdown
↓
Approval Gate
↓
Implementation
↓
Testing
↓
Validation
↓
Report
↓
Done
```

---

REQ-0411  
Coding Agent tidak boleh langsung masuk ke Implementation tanpa Planning.

---

REQ-0412  
Semua langkah harus terdokumentasi di workspace logs.

---

# 43. Code Change Strategy (Diff-Based Execution)

REQ-0420  
Coding Agent tidak boleh menulis ulang file secara penuh tanpa alasan.

---

REQ-0421  
Perubahan kode harus berbasis diff/patch strategy.

---

REQ-0422  
Setiap perubahan harus mencatat:

- file affected
- lines changed
- reason
- impact

---

REQ-0423  
Coding Agent harus bisa rollback perubahan jika gagal test.

---

# 44. File Operation Rules

REQ-0430  
Coding Agent hanya boleh melakukan file operation melalui Tool SDK.

---

REQ-0431  
Coding Agent tidak boleh langsung akses filesystem.

---

REQ-0432  
Semua file create/update/delete harus tercatat di audit log.

---

REQ-0433  
File deletion harus melalui approval gate.

---

# 45. Git Integration Rules

REQ-0440  
Coding Agent harus menggunakan Git untuk semua perubahan.

---

REQ-0441  
Setiap task harus dibuat dalam feature branch.

---

REQ-0442  
Branch naming convention:

```
feature/<agent>/<task-id>
bugfix/<agent>/<task-id>
hotfix/<agent>/<task-id>
```

---

REQ-0443  
Coding Agent tidak boleh merge ke main.

---

REQ-0444  
Merge hanya dilakukan oleh human approval system.

---

# 46. Testing Requirement (Mandatory)

REQ-0450  
Setiap coding task harus menghasilkan atau memperbarui test.

---

REQ-0451  
Coding Agent tidak boleh menandai task selesai tanpa test passing.

---

REQ-0452  
Test wajib mencakup:

- unit test
- integration test (jika relevan)

---

REQ-0453  
Jika test gagal, Coding Agent harus masuk loop:

```
Fix → Retest → Validate
```

---

# 47. Safe Execution Sandbox

REQ-0460  
Coding Agent harus menjalankan kode dalam sandbox environment.

---

REQ-0461  
Tidak boleh menjalankan script berbahaya di host system.

---

REQ-0462  
Semua command execution harus melalui controlled executor.

---

REQ-0463  
Timeout harus diterapkan untuk semua execution (max limit required).

---

# 48. Human Approval Gate

REQ-0470  
Coding Agent harus meminta approval manusia untuk:

- merge ke main
- delete file
- production deployment
- dependency addition

---

REQ-0471  
Approval harus berbasis structured summary:

- what changed
- why changed
- risk level
- affected modules

---

REQ-0472  
Tanpa approval, sistem tidak boleh commit ke protected branch.

---

# 49. Code Quality Enforcement

REQ-0480  
Coding Agent wajib menjalankan linting sebelum menyelesaikan task.

---

REQ-0481  
Backend:

- Ruff
- Black
- MyPy (optional)

---

REQ-0482  
Frontend:

- ESLint
- Prettier
- TypeScript strict mode

---

REQ-0483  
Code tidak boleh merge jika lint error.

---

# 50. Coding Agent Output Format

REQ-0490  
Setiap hasil kerja Coding Agent harus memiliki format:

```json
{
  "task_id": "",
  "status": "success | failed",
  "changes": [],
  "tests": {
    "status": "passed | failed",
    "details": ""
  },
  "summary": "",
  "risk_level": "low | medium | high"
}
```

---

REQ-0491  
Output harus selalu structured, tidak boleh free-form text saja.

---

# END OF PART 5
# PART 6 — SEO AGENT DESIGN

---

# 51. SEO Agent Definition

REQ-0500  
SEO Agent adalah agent yang bertugas menghasilkan, mengoptimasi, dan mengelola konten berbasis SEO untuk website (WordPress atau CMS lain).

---

REQ-0501  
SEO Agent tidak boleh melakukan publish ke production tanpa approval manusia.

---

REQ-0502  
SEO Agent harus bekerja berbasis workflow terstruktur, bukan free-form generation.

---

# 52. SEO Workflow Pipeline

REQ-0510  
SEO Agent wajib mengikuti pipeline berikut:

```
Keyword Research
↓
Intent Analysis
↓
Content Planning
↓
Outline Generation
↓
Article Drafting
↓
SEO Optimization
↓
Internal Linking Suggestions
↓
WordPress Draft
↓
Approval Gate
↓
Publish (optional)
```

---

REQ-0511  
SEO Agent tidak boleh melewati Keyword Research stage.

---

REQ-0512  
Setiap stage harus menghasilkan structured output.

---

# 53. Keyword Research Engine

REQ-0520  
SEO Agent harus mampu menghasilkan keyword berdasarkan:

- niche
- intent
- competition level
- search volume estimation (heuristic allowed)

---

REQ-0521  
Keyword harus diklasifikasikan:

- primary keyword
- secondary keyword
- long-tail keyword

---

REQ-0522  
Keyword tidak boleh dihasilkan secara random tanpa konteks.

---

# 54. Content Planning System

REQ-0530  
SEO Agent harus membuat content plan sebelum menulis artikel.

---

REQ-0531  
Content plan harus mencakup:

- target audience
- search intent
- article structure
- keyword placement strategy

---

REQ-0532  
Tidak boleh langsung generate artikel tanpa plan.

---

# 55. Article Generation Rules

REQ-0540  
Artikel harus berbasis structured outline.

---

REQ-0541  
Artikel harus memiliki:

- H1
- H2
- H3 (jika diperlukan)
- meta description
- slug suggestion

---

REQ-0542  
Keyword harus natural, tidak boleh keyword stuffing.

---

REQ-0543  
Artikel harus SEO-friendly tapi tetap human-readable.

---

# 56. SEO Optimization System

REQ-0550  
SEO Agent harus melakukan optimization setelah artikel dibuat.

---

REQ-0551  
Optimasi mencakup:

- keyword density check
- readability improvement
- heading structure validation
- meta tag optimization

---

REQ-0552  
SEO Agent harus memberikan SEO score (0–100).

---

REQ-0553  
Artikel dengan score di bawah threshold harus direvisi.

---

# 57. Internal Linking System

REQ-0560  
SEO Agent harus menyarankan internal link antar artikel.

---

REQ-0561  
Internal linking harus berbasis:

- semantic similarity
- topic relevance
- keyword overlap

---

REQ-0562  
Tidak boleh membuat link random tanpa konteks.

---

# 58. WordPress Integration Rules

REQ-0570  
SEO Agent dapat membuat draft artikel ke WordPress.

---

REQ-0571  
SEO Agent tidak boleh publish langsung tanpa approval.

---

REQ-0572  
WordPress draft harus mencakup:

- title
- slug
- content
- categories
- tags
- meta description

---

REQ-0573  
Integration harus melalui API layer, bukan direct DB access.

---

# 59. SEO Memory System

REQ-0580  
SEO Agent harus menyimpan data:

- keyword history
- published articles
- ranking performance (manual input allowed)
- content templates

---

REQ-0581  
Memory harus digunakan untuk meningkatkan kualitas artikel berikutnya.

---

REQ-0582  
SEO Agent harus belajar dari artikel sebelumnya dalam workspace.

---

# 60. SEO Quality Control

REQ-0590  
SEO Agent harus memvalidasi output sebelum dianggap selesai:

- grammar check
- SEO score threshold (>75)
- readability check
- duplication check

---

REQ-0591  
Jika tidak memenuhi standar, agent harus masuk revision loop:

```
Draft → Review → Optimize → Recheck
```

---

# END OF PART 6
# PART 7 — DEPLOYMENT & SYSTEM OPERATION

---

# 61. Deployment Philosophy

REQ-0600  
Agent Sky harus dapat dijalankan sepenuhnya di mesin lokal (home PC) menggunakan Docker Compose.

---

REQ-0601  
Sistem tidak bergantung pada cloud hosting wajib.

---

REQ-0602  
Cloud hanya optional untuk akses remote (via tunnel).

---

REQ-0603  
Seluruh system harus restart-safe (bisa hidup kembali setelah crash tanpa manual intervention).

---

# 62. Docker System Architecture

REQ-0610  
Semua service harus berjalan dalam Docker Compose.

---

REQ-0611  
Minimal services:

- backend (FastAPI)
- frontend (Next.js)
- postgres
- redis

---

REQ-0612  
Semua service harus memiliki healthcheck endpoint.

---

REQ-0613  
Service failure harus otomatis restart.

---

REQ-0614  
Tidak boleh ada service yang berjalan di luar Docker dalam production mode.

---

# 63. Local PC Always-On System

REQ-0620  
Agent Sky harus dirancang untuk berjalan 24/7 di PC pribadi.

---

REQ-0621  
System harus mampu recover dari:

- power failure
- crash
- docker restart
- network disconnect

---

REQ-0622  
State harus disimpan di database, bukan memory runtime.

---

REQ-0623  
System harus memiliki auto-start saat machine boot.

---

# 64. Cloudflare Tunnel Integration

REQ-0630  
Remote access ke Agent Sky harus menggunakan Cloudflare Tunnel.

---

REQ-0631  
Tidak boleh expose port langsung ke internet.

---

REQ-0632  
Tunnel digunakan untuk:

- frontend dashboard access
- API remote control
- webhook integration (optional)

---

REQ-0633  
Tunnel harus secure dengan authentication layer.

---

# 65. Telegram Control System

REQ-0640  
Agent Sky harus dapat dikontrol via Telegram bot.

---

REQ-0641  
Telegram bot dapat:

- trigger agent run
- check system status
- view logs summary
- approve/reject tasks

---

REQ-0642  
Telegram tidak boleh bisa akses raw system command.

---

REQ-0643  
Semua command Telegram harus melewati API validation layer.

---

# 66. Logging & Monitoring System

REQ-0650  
Semua system activity harus dicatat dalam structured logs.

---

REQ-0651  
Log harus mencakup:

- timestamp
- service
- event type
- request_id
- agent_id (if any)
- status

---

REQ-0652  
Log harus bisa di-query oleh backend API.

---

REQ-0653  
System harus memiliki basic monitoring dashboard.

---

# 67. Health Check System

REQ-0660  
Setiap service wajib memiliki endpoint /health.

---

REQ-0661  
Health status:

- healthy
- degraded
- unhealthy

---

REQ-0662  
Backend harus dapat mengumpulkan status semua service.

---

REQ-0663  
Jika service unhealthy, system harus trigger alert event.

---

# 68. Backup & Recovery System

REQ-0670  
Database harus di-backup secara periodik.

---

REQ-0671  
Backup harus mencakup:

- PostgreSQL dump
- Redis snapshot (optional)
- workspace data

---

REQ-0672  
Backup harus bisa restore otomatis.

---

REQ-0673  
System harus memiliki rollback capability untuk deployment terakhir.

---

# 69. Production vs Development Mode

REQ-0680  
System harus mendukung dua mode:

- development
- production

---

REQ-0681  
Development mode:

- debug enabled
- hot reload
- relaxed validation

---

REQ-0682  
Production mode:

- strict validation
- logging minimal sensitive data
- no debug endpoints

---

REQ-0683  
Mode harus ditentukan via environment variable.

---

# 70. System Startup & Shutdown Rules

REQ-0690  
System startup harus mengikuti urutan:

```
Database
↓
Redis
↓
Backend API
↓
Core Engine
↓
Agent Runtime
↓
Frontend
```

---

REQ-0691  
Shutdown harus graceful:

- stop accepting new jobs
- finish running jobs
- save state
- shutdown services

---

REQ-0692  
Forced shutdown harus tetap trigger state save jika memungkinkan.

---

# END OF PART 7
# PART 8 — ROADMAP & PRODUCT EVOLUTION

---

# 71. Versioning Philosophy

REQ-0700  
Agent Sky harus menggunakan semantic versioning:

- v0.x → development stage
- v1.0 → stable core system
- v2.0+ → scaling & advanced automation

---

REQ-0701  
Setiap major version harus memiliki milestone dokumentasi lengkap.

---

REQ-0702  
Tidak boleh ada breaking change tanpa version bump.

---

# 72. Sprint System

REQ-0710  
Semua development harus berbasis sprint kecil.

---

REQ-0711  
Satu sprint hanya boleh memiliki satu fokus utama.

Contoh:

- Sprint 001 → Bootstrap
- Sprint 002 → Workspace
- Sprint 003 → AI Provider

---

REQ-0712  
Tidak boleh ada multi-goal sprint.

---

REQ-0713  
Sprint harus selalu menghasilkan:

- working system
- test coverage
- documentation update

---

# 73. Feature Prioritization System

REQ-0720  
Semua fitur harus melalui prioritas:

- P0 (core system)
- P1 (important)
- P2 (nice to have)
- P3 (future idea)

---

REQ-0721  
P0 fitur harus selesai sebelum P1 dimulai.

---

REQ-0722  
Tidak boleh mengerjakan fitur P2 atau P3 sebelum P0 stabil.

---

# 74. Scaling Strategy

REQ-0730  
System harus dirancang untuk scaling horizontal di masa depan.

---

REQ-0731  
Semua service harus stateless jika memungkinkan.

---

REQ-0732  
State harus dipindahkan ke:

- PostgreSQL
- Redis
- Workspace storage

---

REQ-0733  
Core Engine harus bisa dipisah menjadi microservice di masa depan tanpa redesign total.

---

# 75. Future Agent Expansion

REQ-0740  
Agent Sky harus mendukung penambahan agent tanpa perubahan core system.

---

REQ-0741  
Agent baru harus mengikuti standard interface:

- goal input
- plan output
- execution lifecycle
- tool usage
- memory access

---

## Planned Future Agents:

REQ-0742  

- Coding Agent (P0)
- SEO Agent (P0)
- WordPress Agent (P1)
- Telegram Agent (P1)
- Automation Agent (P1)
- Photo Editing Agent (P2)
- Business Intelligence Agent (P2)
- Finance Agent (P3)
- Personal Assistant Agent (P3)

---

# 76. Plugin System Vision

REQ-0750  
Agent Sky harus memiliki plugin system untuk tools dan agents.

---

REQ-0751  
Plugin harus bisa:

- install
- enable/disable
- update
- remove

tanpa mengubah core system.

---

REQ-0752  
Plugin harus sandboxed.

---

# 77. Observability & Evolution Tracking

REQ-0760  
System harus bisa melacak:

- agent performance
- task success rate
- system bottleneck
- execution time

---

REQ-0761  
Data observability harus digunakan untuk meningkatkan system secara iteratif.

---

REQ-0762  
System harus self-improving secara manual (human-driven optimization).

---

# 78. Roadmap Structure

REQ-0770  
Roadmap harus dibagi menjadi:

### Phase 1 — Foundation
- Docker
- Backend
- Frontend
- Core Engine

---

### Phase 2 — Intelligence Layer
- AI Provider
- Memory System
- Agent Runtime

---

### Phase 3 — Agents
- Coding Agent
- SEO Agent
- WordPress Agent

---

### Phase 4 — Automation
- Telegram
- Scheduler
- Automation Agent

---

### Phase 5 — Expansion
- Plugin system
- Advanced agents
- Scaling

---

# 79. Release Strategy

REQ-0780  
Release harus melalui staging environment sebelum production.

---

REQ-0781  
Tidak boleh deploy langsung ke production dari feature branch.

---

REQ-0782  
Setiap release harus memiliki:

- changelog
- test report
- risk analysis

---

# 80. Long Term Vision

REQ-0790  
Agent Sky bukan hanya software, tetapi:

> "Personal AI Operating System"

---

REQ-0791  
System harus bisa berkembang menjadi:

- AI development environment
- automation engine
- content generation system
- personal assistant layer

---

REQ-0792  
Semua pengembangan harus tetap backward compatible dengan Manifest.

---

# END OF PART 8
# PART 9 — DEVELOPMENT RULES FINAL & CONSTITUTION

---

# 81. System Hierarchy of Truth

REQ-0800  
Jika terjadi konflik antara:

- chat instruction
- code implementation
- documentation
- Manifest

Maka yang selalu benar adalah:

> AGENT_SKY_MANIFEST.md

---

REQ-0801  
Tidak ada exception untuk rule ini kecuali diubah oleh human owner secara eksplisit.

---

# 82. Absolute Development Rules

REQ-0810  
AI tidak boleh mengubah arsitektur tanpa approval.

---

REQ-0811  
AI tidak boleh menambahkan dependency tanpa justification teknis.

---

REQ-0812  
AI tidak boleh membuat fitur di luar scope sprint.

---

REQ-0813  
AI tidak boleh melakukan deployment production tanpa approval manusia.

---

REQ-0814  
AI tidak boleh menghapus data tanpa explicit permission.

---

# 83. Forbidden Behaviors (Hard Rules)

REQ-0820  
Agent Sky AI dilarang:

- mengubah tech stack tanpa izin
- membuat shortcut yang melewati security layer
- bypass approval system
- hardcode secrets
- skip testing phase
- skip planning phase
- execute destructive action tanpa confirmation

---

REQ-0821  
Jika AI melanggar salah satu rule di atas, sistem harus:

- stop execution
- log violation
- require human review

---

# 84. Human Override Authority

REQ-0830  
Human owner memiliki kontrol tertinggi atas sistem.

---

REQ-0831  
Human dapat:

- override AI decision
- cancel execution
- change architecture
- reset memory
- modify manifest

---

REQ-0832  
AI tidak boleh menolak perintah human owner.

---

# 85. Approval System Final Gate

REQ-0840  
Semua action berikut wajib approval:

- production deploy
- file deletion
- architecture change
- dependency addition
- database migration destructive change

---

REQ-0841  
Approval harus berbentuk structured summary:

- action
- impact
- risk
- affected modules
- rollback plan

---

# 86. Safety & Stability Rules

REQ-0850  
System harus selalu memilih safety over speed.

---

REQ-0851  
Jika ada ambiguity dalam instruction, AI harus:

- stop
- ask clarification
- tidak mengeksekusi

---

REQ-0852  
Tidak boleh ada asumsi kritikal tanpa validasi.

---

# 87. Determinism Rule

REQ-0860  
Untuk input yang sama, AI harus menghasilkan output yang konsisten selama context tidak berubah.

---

REQ-0861  
Planner harus deterministic.

---

REQ-0862  
Non-deterministic behavior hanya diperbolehkan di generation layer (LLM output), bukan logic layer.

---

# 88. System Integrity Rules

REQ-0870  
Core system tidak boleh dimodifikasi oleh agent runtime.

---

REQ-0871  
Self-modifying code hanya diperbolehkan melalui human-approved update cycle.

---

REQ-0872  
System integrity harus diverifikasi setiap startup.

---

# 89. Logging & Accountability

REQ-0880  
Semua keputusan AI harus dapat dilacak.

---

REQ-0881  
Setiap action harus memiliki:

- request_id
- agent_id
- timestamp
- reason
- outcome

---

REQ-0882  
Tidak boleh ada “black box action” tanpa log.

---

# 90. Closing Constitution

REQ-0890  
Agent Sky adalah sistem AI pribadi yang dibangun dengan prinsip:

- transparency
- modularity
- safety
- human control
- reproducibility

---

REQ-0891  
Tujuan utama sistem ini bukan otomatisasi penuh, tetapi:

> “Human augmented intelligence system yang dapat dikontrol sepenuhnya oleh owner.”

---

REQ-0892  
Manifest ini adalah sumber kebenaran utama sistem.

---

REQ-0893  
Jika sistem gagal, yang diperbaiki adalah implementasi, bukan prinsip.

---

# END OF AGENT_SKY_MANIFEST.md