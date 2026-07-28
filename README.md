# DocMind AI

An AI-powered RAG (Retrieval-Augmented Generation) document assistant — upload PDF/DOCX files and ask questions grounded only in their content, with page-level citations and real-time streaming answers.

This is the **meta-repository**. It ties together two independent projects as git submodules and provides the local development infrastructure shared by both:

| Repo | Description |
|---|---|
| [`backend-DocMind AI`](https://github.com/ChamathDilshanC/backend-DocMind-AI) | .NET 8 Web API — Clean Architecture + CQRS (MediatR), Semantic Kernel RAG pipeline, Qdrant vector search, PostgreSQL, SignalR streaming, Hangfire background jobs, Redis cache, JWT + Google OAuth. |
| [`frontend-DocMind AI`](https://github.com/ChamathDilshanC/frontend-DocMind-AI) | Next.js 16 (App Router) + React 19 — chat UI with token streaming and inline citations, document upload with live status, Google Sign-In. |

Full architecture spec: [`docs/RAG System Architecture.pdf`](docs/RAG%20System%20Architecture.pdf).

## Architecture at a glance

```
React/Next.js client
        │  REST + SignalR
        ▼
.NET 8 Web API  ──►  Auth / Documents / Chat services
        │
        ▼
Semantic Kernel  ──►  OpenAI (chat + embeddings)
        │
        ▼
Qdrant (vector search)  +  PostgreSQL (relational data)  +  Redis (cache)  +  Hangfire (background jobs)
```

Document flow: upload → extract text (PdfPig / OpenXml) → clean → chunk (500 words / 100-word overlap) → embed → store in Qdrant, all as an async Hangfire job with live progress over SignalR.

Chat flow: question → embed → Qdrant similarity search (top 5) → prompt assembly → streamed LLM completion → answer + citations, persisted to conversation history.

## Getting started

**1. Clone with submodules**

```bash
git clone --recurse-submodules https://github.com/ChamathDilshanC/Main-DocMind-AI.git
# or, if already cloned without submodules:
git submodule update --init --recursive
```

**2. Start local infrastructure** (PostgreSQL, Qdrant, Redis)

```bash
cp .env.example .env
docker compose up -d
```

**3. Run the backend**

```bash
cd "backend-DocMind AI"
# See appsettings.README.md for required secrets (OpenAI key, JWT signing key, Google OAuth Client ID)
dotnet run --project src/DocumentAssistant.API
```

The API applies EF Core migrations automatically on startup in the `Development` environment. Swagger UI: `https://localhost:7045/swagger` (or the HTTP port shown in the console).

**4. Run the frontend**

```bash
cd "frontend-DocMind AI"
npm install
cp .env.example .env.local   # fill in NEXT_PUBLIC_GOOGLE_CLIENT_ID
npm run dev
```

Open `http://localhost:3000`.

## Scope

This implementation covers Phases 1–4 of the architecture doc, built to production quality:

- **Phase 1**: Authentication (JWT + refresh token rotation, Google OAuth), user management, file upload, PostgreSQL setup.
- **Phase 2**: PDF/DOCX parsing, chunking, embedding generation, Qdrant integration.
- **Phase 3**: Semantic Kernel RAG pipeline, chat interface, SignalR streaming.
- **Phase 4**: Conversation history, citations, Redis caching, Hangfire background jobs.

### Roadmap (not built in this pass)

**Phase 5**: OCR, multi-document search polish, admin dashboard UI, Docker/Kubernetes production deployment, monitoring.

**Future features**: image understanding, OCR for scanned PDFs, voice chat, multi-language search, agentic workflows, email document analysis, knowledge graph integration, Teams/Slack/WhatsApp bots, mobile app.

Also explicitly deferred: virus scanning on uploads (flagged in the backend's security notes as a future improvement, not stubbed).

## License

MIT — see [LICENSE](LICENSE).
