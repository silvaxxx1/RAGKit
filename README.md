# 🧠 RAGKit

**A modular, production-oriented foundation for building Retrieval-Augmented Generation (RAG) systems.**

RAGKit takes you from ingestion → parsing → chunking → embeddings → vector storage → retrieval → generation, without locking you into a single vector database, embedding provider, or LLM.

It's built to be the project you clone when you're starting a real RAG system, not a toy demo.

---

<p align="center">
  <img src="ragapp.png" alt="RAGKit Architecture" width="700"/>
</p>

---

## Why RAGKit?

Most RAG projects start as a tightly coupled chain:

```text
PDF → LangChain → Vector DB → OpenAI
```

That works for a prototype, but becomes hard to extend as requirements grow — different vector stores per environment, different embedding models, OCR for scanned documents, multimodal inputs, custom chunking rules.

RAGKit separates these concerns behind explicit interfaces so each piece can be swapped, extended, or replaced independently.

```text
                    RAGKit
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
    Ingestion      Retrieval      Generation
        │              │              │
     Parser         Vector DB          LLM
     Chunker        PGVector          OpenAI
     Embeddings     Qdrant            Cohere
     OCR                              Ollama
```

---

## ✅ Status — v2.0.0

* **Vector storage**: fully migrated to PostgreSQL + PGVector (MongoDB removed). Qdrant remains supported as a second backend — switch between them with one `.env` variable.
* **Chunking**: custom token-aware chunker, no LangChain dependency required (LangChain chunking can still be enabled optionally).
* **Pipelines**: async ingestion and retrieval paths, reworked for throughput.
* **Backend**: stable and deployable; Celery + Redis for distributed background processing is the next milestone.

---

## 🧰 Toolbox (in progress)

The goal is for RAGKit to be a configurable toolbox you start production RAG projects from — not just a vector-DB wrapper. Planned additions, following the same provider-interface pattern used for LLMs and vector stores:

* **Pluggable chunking strategies** — fixed-size, semantic, recursive, and document-structure-aware chunkers selectable via config, instead of one hardcoded strategy.
* **Out-of-the-box OCR** — ingest scanned PDFs and images directly into the pipeline, with OCR as a swappable provider (e.g. Tesseract, cloud OCR APIs).
* **Multimodal support** — image + text embeddings and multimodal-capable LLMs, so ingestion isn't limited to plain text documents.

These will land as new provider interfaces under `stores/`, configured the same way vector DBs and LLMs are today — no changes to application logic required to adopt them.

---

## ✨ Features

### 🧩 Modular Architecture
Provider implementations are isolated from application logic — swap components without touching the pipeline.

### 🗄️ Multiple Vector Stores
* PostgreSQL + PGVector (primary)
* Qdrant

```env
VECTOR_DB_PROVIDER=pgvector   # or qdrant
```

### 🤖 Multiple LLM Providers
* OpenAI
* Cohere
* Ollama

### 🔢 Multiple Embedding Providers
* OpenAI
* Cohere
* Sentence Transformers

### ✂️ Custom Chunking
Token-aware chunking built for the pipeline, with optional LangChain fallback.

### ⚡ Async-First Backend
FastAPI + Uvicorn, built so heavier ingestion workloads can move to dedicated workers later.

### 🐳 Dockerized Infrastructure
PostgreSQL, PGVector, and Qdrant all runnable via Docker Compose.

### 🧪 Testing
```text
79 tests passing
```
Runs without requiring the full infrastructure stack.

### 📊 Observability
Prometheus metrics (`rag_query_latency_seconds`, `retrieval_hit_rate`, etc.), kept separate from core RAG logic.

---

## 🏗️ Architecture

```text
                Documents
                    │
                    ▼
              ┌───────────┐
              │  Parser   │  (OCR planned)
              └─────┬─────┘
                    │
                    ▼
              ┌───────────┐
              │  Chunker  │  (pluggable strategies planned)
              └─────┬─────┘
                    │
                    ▼
             ┌────────────┐
             │ Embeddings │  (multimodal planned)
             └──────┬─────┘
                    │
                    ▼
        ┌──────────────────────┐
        │     Vector Store     │
        │  PGVector | Qdrant   │
        └───────────┬──────────┘
                    │
                    ▼
                 Retrieval
                    │
                    ▼
              ┌───────────┐
              │    LLM    │
              └─────┬─────┘
                    │
                    ▼
                 Response
```

---

## 📦 Tech Stack

| Layer              | Technology                              |
| ------------------ | ---------------------------------------- |
| API                | FastAPI + Uvicorn                        |
| Language           | Python                                   |
| Package Management | uv                                       |
| Vector Store       | PostgreSQL + PGVector / Qdrant           |
| Embeddings         | OpenAI / Cohere / Sentence Transformers  |
| LLM                | OpenAI / Cohere / Ollama                 |
| Chunking           | Custom token-aware chunker               |
| Database           | PostgreSQL                               |
| Infrastructure     | Docker / Docker Compose                  |
| Observability      | Prometheus                               |
| Testing            | Pytest                                   |

---

## 🚀 Quickstart

### 1. Clone
```bash
git clone https://github.com/silvaxxx1/RagApp.git
cd RagApp
```

### 2. Install dependencies
```bash
uv init
uv add -r requirements.txt
```

### 3. Configure environment
```bash
cp uv.example .env
```
Set your API keys and choose providers:
```env
VECTOR_DB_PROVIDER=pgvector   # or qdrant
EMBEDDING_PROVIDER=openai
LLM_PROVIDER=openai
```

### 4. Start infrastructure
```bash
cd docker
docker-compose up -d
```

### 5. Run the backend
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 5000
```
Swagger UI → [http://localhost:5000/docs](http://localhost:5000/docs)

### 6. Run tests
```bash
pytest
```

---

## 🧱 Design Principles

1. **Separation of concerns** — the RAG pipeline shouldn't depend on a specific database or model provider.
2. **Replaceable infrastructure** — swap providers without rewriting business logic.
3. **Explicit abstractions** — every core component has a clear interface.
4. **Configuration over hard-coding** — provider selection lives in `.env`, not code.
5. **Production-oriented foundations** — patterns that hold up moving from local dev to production.

---

## 🛣️ Roadmap

**Toolbox**
- [ ] Pluggable chunking strategies (semantic, recursive, structure-aware)
- [ ] Out-of-the-box OCR ingestion
- [ ] Multimodal embeddings and LLMs

**Infrastructure**
- [ ] Celery + Redis background workers
- [ ] Additional vector-store providers
- [ ] Kubernetes deployment templates
- [ ] CI/CD examples

**Retrieval**
- [ ] Hybrid dense + BM25 retrieval
- [ ] Reranking
- [ ] Multi-query / parent-child retrieval

**Evaluation**
- [ ] RAGAS integration
- [ ] Golden datasets and latency benchmarking

---

## 🤝 Contributing

Contributions are welcome — new vector-store providers, embedding/LLM providers, chunking strategies, OCR backends, retrieval strategies, or documentation.

The preferred pattern is to extend an existing interface rather than add provider-specific logic into the application layer.

---

## 📄 License

MIT License — see [LICENSE](./LICENSE)
