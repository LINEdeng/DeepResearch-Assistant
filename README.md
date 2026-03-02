# DeepResearch Assistant

An AI-powered deep research platform for industry intelligence. It supports smart search, knowledge graphs, data visualization, and—uniquely—**two agent reasoning modes**: ReAct (single-agent) and Multi-Agent LangGraph (six-role pipeline).

---

## Features

| Feature | Description |
|--------|-------------|
| **Deep Research** | Run full research workflows with planning, search, analysis, writing, and review. |
| **Dual Agent Modes** | Choose **ReAct** (reasoning + acting, one agent) or **Multi-Agent** (specialized roles, LangGraph). |
| **Web + Local Search** | Web search (Bocha API) and local knowledge base (RAG over uploaded docs). |
| **Knowledge Graph** | Build and visualize entity-relation graphs from research. |
| **Charts & Reports** | Auto-generated charts and structured reports with references. |
| **Chat** | Conversational QA with optional chain-of-thought (e.g. DeepSeek R1). |
| **Sessions & Checkpoints** | Resume long research from saved checkpoints. |
| **Auth & Data** | JWT auth, PostgreSQL, Redis, Milvus, Elasticsearch, MinIO. |

---

## Two Deep-Thinking Modes

The project supports two distinct agent architectures for deep research. You can use one or the other depending on whether you prefer a single reasoning agent or a multi-role pipeline.

### Mode 1: ReAct (Reasoning + Acting)

One agent that **reasons step-by-step** and **acts** by calling tools (search, knowledge base, Text2SQL, charts, etc.). Thought process is visible as chain-of-thought.

**Flow:**

```
┌─────────┐     ┌──────────────┐     ┌─────────────┐     ┌────────────┐
│  Plan   │ ──► │   Execute    │ ──► │   Reflect   │ ──► │ Synthesize │
│ (LLM)   │     │ (parallel    │     │ (evaluate   │     │ (report)   │
│         │     │  search)     │     │  coverage)   │     │             │
└─────────┘     └──────────────┘     └─────────────┘     └────────────┘
       │                  │                   │
       │                  │                   └──► loop: more search if needed
       ▼                  ▼
  Sub-queries       Web / KB / DB / Charts
```

- **Plan**: LLM breaks the question into sub-queries and strategy.
- **Execute**: Run multiple searches in parallel (web, knowledge base, etc.).
- **Reflect**: Check if information is sufficient; if not, add more queries and repeat.
- **Synthesize**: Produce the final report.

**Agent flow (single agent):**

```
User Query → [ReAct Agent] ⇄ Think → Act (tools) → Observe → … → Finish
                              ↑_____________________________|
```

Tools available: `web_search`, `knowledge_search`, `text2sql`, `data_analyzer`, `chart_generator`, `stock_query`, `bidding_search`, `finish`.

---

### Mode 2: Multi-Agent (LangGraph Pipeline)

Six **specialized agents** in a fixed pipeline: Plan → Research → Analyze → Write → Review → (optionally) Revise → Complete.

**Flow:**

```
                    ┌─────────────────────────────────────────────────────────────┐
                    │                    Multi-Agent Pipeline                       │
                    └─────────────────────────────────────────────────────────────┘
  User Query
       │
       ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│    Plan      │───►│   Research   │───►│   Analyze    │───►│    Write     │
│ ChiefArchitect│   │  DeepScout   │    │ CodeWizard   │    │  LeadWriter  │
│ (outline)    │    │ (web + KB)   │    │ (charts, KG) │    │ (draft)      │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                   │
       ┌───────────────────────────────────────────────────────────┘
       ▼
┌──────────────┐      need revise?      ┌──────────────┐
│   Review     │ ──── yes ────────────► │   Revise     │
│ CriticMaster │                        │  LeadWriter  │
│ (critique)   │ ◄─────────────────────│ (revision)   │
└──────┬───────┘                        └──────────────┘
       │ no
       ▼
    Complete
```

**Agents:**

| Step | Agent | Role |
|------|--------|------|
| 1 | **ChiefArchitect** | Understands the query and produces a research outline. |
| 2 | **DeepScout** | Runs deep search (web and/or local KB), gathers facts. |
| 3 | **CodeWizard** / **DataAnalyst** | Builds charts, knowledge graph, and data views. |
| 4 | **LeadWriter** | Writes the first draft from outline + facts + charts. |
| 5 | **CriticMaster** | Reviews the draft; flags gaps, missing sources, clarity. |
| 6 | **LeadWriter** (Revise) | Revises if the critic requests changes; then back to Review. |

The graph uses **conditional edges**: after Review, the flow goes to **Revise** if there are unresolved critical issues (within max iterations), otherwise to **Complete**.

---

## Tech Stack

- **Backend**: Python 3.10+, FastAPI, LangGraph (multi-agent), OpenAI-compatible LLM API (e.g. DashScope).
- **Frontend**: React, Vite.
- **Data**: PostgreSQL, Redis, Milvus, Elasticsearch, MinIO (Docker Compose).

---

## Quick Start

### 1. Clone and start services

```bash
git clone <your-repo-url>
cd industry_information_assistant

# Start PostgreSQL, Redis, Milvus, etc.
./start-services.sh start
# or: docker compose up -d
```

### 2. Backend

```bash
cd backend
cp .env.example .env
# Set DASHSCOPE_API_KEY, BOCHA_API_KEY, POSTGRES_*, JWT_SECRET_KEY, etc.

pip install -r requirements.txt
python app/app_main.py
```

API: `http://localhost:8000`, docs: `http://localhost:8000/docs`.

### 3. Frontend

```bash
cd frontend
npm install --legacy-peer-deps
npm run dev
```

App: `http://localhost:5173`.

### 4. Deep research (streaming)

```bash
curl -N -X POST "http://localhost:8000/research/stream" \
  -H "Content-Type: application/json" \
  -d '{"query": "Your research question here", "session_id": "<uuid>", "search_web": true}'
```

---

## Project Structure

```
industry_information_assistant/
├── backend/
│   ├── app/
│   │   ├── service/
│   │   │   ├── deep_research_v2/   # Multi-Agent (LangGraph) pipeline
│   │   │   │   ├── graph.py        # Plan→Research→Analyze→Write→Review→Revise
│   │   │   │   └── agents/         # ChiefArchitect, DeepScout, CodeWizard, etc.
│   │   │   ├── dr_g.py             # ReAct research service
│   │   │   └── react_controller.py # ReAct loop: Plan→Execute→Reflect→Synthesize
│   │   ├── router/                 # research, chat, documents, auth, ...
│   │   └── app_main.py
│   ├── docker-compose-base.yml
│   └── requirements.txt
├── frontend/
│   └── src/
└── README.md
```

---

## Environment

| Variable | Purpose |
|----------|---------|
| `DASHSCOPE_API_KEY` | LLM & embedding (e.g. Aliyun Bailian). |
| `BOCHA_API_KEY` | Web search API. |
| `POSTGRES_*` | PostgreSQL connection. |
| `JWT_SECRET_KEY` | Auth signing. |

See `backend/.env.example` and the root `READMED.md` (Chinese) for full options.

---

## API Docs

After starting the backend: **http://localhost:8000/docs**

---

## License

See repository license file.
