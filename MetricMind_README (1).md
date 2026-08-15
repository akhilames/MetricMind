# MetricMind — Agentic Semantic BI Engine

> Ask "Why did our European margins drop last quarter?" in plain English and get back a governed, trustworthy answer — not a hallucinated SQL query.

An LLM agent that is **structurally unable** to hallucinate a business number: its only tool calls a governed semantic layer (Cube.dev), which resolves against one dbt-owned formula, which lives in one warehouse table.

```
Next.js Chat UI  →  FastAPI + LangChain Agent  →  Cube.dev Semantic Layer  →  dbt mart  →  Warehouse
```

Full architecture: [`docs/architecture.md`](docs/architecture.md)

## Repository layout

```
metricmind/
├── dbt/                  # Governed data modeling (raw → staging → mart)
│   ├── seeds/raw_transactions.csv   # mock Q3 vs Q4 2025 data — margins really do drop
│   ├── models/staging/stg_transactions.sql
│   └── models/marts/fct_revenue_margin.sql   # the single source of truth for margin
├── semantic-layer/       # Cube.dev — the ONLY thing the agent is allowed to query
│   ├── cube.js
│   └── schema/{Revenue,Margin}.js
├── agent/                 # LangChain orchestrator + FastAPI wrapper
│   ├── orchestrator.py
│   ├── app.py
│   ├── tools/semantic_api_tool.py   # governance allow-list lives here
│   └── prompts/system_prompt.txt
├── web/                   # Next.js chat UI
│   └── app/chat, components/ChatWindow.tsx
├── tests/
│   ├── governance_audit.py          # same question -> same number
│   └── api_translation_tests.py     # governance boundary unit tests
├── docker-compose.yml
└── Makefile
```

## Quickstart (local, no cloud warehouse needed)

This repo runs entirely on your machine using **DuckDB** as the warehouse —
swap in Snowflake/Databricks later by changing one dbt target.

```bash
git clone https://github.com/<your-username>/metricmind.git
cd metricmind
cp .env.example .env               # add your ANTHROPIC_API_KEY or OPENAI_API_KEY

# 1) Build the governed marts (verified working — DuckDB, zero setup)
cd dbt && pip install dbt-duckdb
DBT_PROFILES_DIR=. dbt seed
DBT_PROFILES_DIR=. dbt run
DBT_PROFILES_DIR=. dbt test
cd ..

# 2) Start the semantic layer (Cube.dev), pointed at the DuckDB file above
cd semantic-layer && cp .env.example .env && npm install && npm run dev
# → http://localhost:4000

# 3) Start the agent API (new terminal)
cd agent && pip install -r requirements.txt
uvicorn app:app --reload --port 8000
# → http://localhost:8000/chat

# 4) Start the chat UI (new terminal)
cd web && cp .env.local.example .env.local && npm install && npm run dev
# → http://localhost:3000/chat
```

Or with Docker: `docker compose up` (run the `dbt` step above once first, so
`metricmind.duckdb` exists before `semantic-layer` starts).

Then open `http://localhost:3000/chat` and ask:
*"Why did our European margins drop last quarter?"*

## What's already verified to run

- ✅ `dbt seed && dbt run && dbt test` — builds `fct_revenue_margin` on DuckDB, all tests pass
- ✅ The mock dataset genuinely shows Europe's gross margin falling **46.07% → 38.45%**
  quarter-over-quarter, driven by shipping cost, not cost of goods — a real story for the agent to find
- ✅ `python -m pytest tests/api_translation_tests.py -v` — governance boundary rejects any
  ungoverned measure/dimension before it can reach the semantic layer
- ⚙️ The Cube.dev semantic layer, LangChain agent, and Next.js UI are complete, wired code —
  running them end-to-end additionally needs `npm install` for Cube/Next.js and an LLM API key

## Governance audit

```bash
make audit   # proves: same question -> same number, every time, no LLM involved
make test    # unit tests for the governance allow-list
```

## License

MIT — free to use for learning and portfolio purposes.
