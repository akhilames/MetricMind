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
