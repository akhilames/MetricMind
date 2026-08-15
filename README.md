# MetricMind
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
