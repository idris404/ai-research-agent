# LORE

*Language-based Open Research Engine*

<p align="center"><strong>AI research on autopilot. 40 papers. Every monday. Zero manual work.</strong></p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/FastAPI-0.115-009688?style=flat-square&logo=fastapi&logoColor=white" />
  <img src="https://img.shields.io/badge/Qdrant-1.12-red?style=flat-square" />
  <img src="https://img.shields.io/badge/Docker-compose-2496ED?style=flat-square&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-22C55E?style=flat-square" />
</p>

---

## Why

Keeping up with AI research means reading dozens of papers a week, manually. LORE automates the entire loop: it fetches papers from arXiv and trending repos from GitHub, embeds them into a local vector store, runs a RAG synthesis with a Groq-hosted LLM, and pushes a structured digest to three Notion databases every week. One cron trigger in n8n is all it takes.

---

## Quick Start

```bash
git clone https://github.com/[username]/lore
cd lore
cp .env.example .env
docker-compose up --build
```

Fill in `.env` before starting. Required keys: `GROQ_API_KEY`, `NOTION_TOKEN`, and the three Notion database IDs. See `.env.example` for all variables.

---

## How it works

| Step | What happens |
|---|---|
| Ingest | Fetches papers from arXiv (cs.AI, cs.LG, cs.MA) and repos from GitHub, filtered by keywords and date range |
| Embed | Encodes title + abstract with `all-MiniLM-L6-v2` (dim=384) and upserts vectors into Qdrant |
| Synthesize | Runs semantic search over the collection, feeds top-K results to LLaMA 3.3 70B via LangChain, returns a structured synthesis with cited sources |
| Detect | Counts keyword and category frequency across the collection to surface emerging topics |

---

## Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/ingest/` | Fetch and embed papers from arXiv and GitHub |
| POST | `/ingest/single` | Embed and store a single document |
| GET | `/search/` | Semantic similarity search with optional source filter |
| POST | `/synthesize/` | RAG synthesis over top-K retrieved documents |
| GET | `/trends/` | Keyword and category frequency across the collection |
| POST | `/digest/run` | Full pipeline: ingest, synthesize, push to Notion |

---

## n8n Integration

```bash
# Connect n8n to the FastAPI Docker network
docker network connect ai-research-agent_default <n8n-container-name>

# Weekly digest — HTTP Request node
POST http://fastapi:8000/digest/run

# On-demand ingest — HTTP Request node
POST http://fastapi:8000/ingest/
```

If n8n runs locally outside Docker, replace `fastapi` with `localhost`.

---

## Stack

| Layer | Technology |
|---|---|
| API | FastAPI + Uvicorn |
| Vector store | Qdrant (local, Docker) |
| Embeddings | Sentence Transformers `all-MiniLM-L6-v2` |
| LLM | Groq API, LLaMA 3.3 70B via LangChain |
| Automation | n8n (external) |
| Output | Notion API |

---

## License

MIT
