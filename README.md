# CorridorSync

Crowd-powered commute intelligence platform for Bengaluru.  
Built for the **IBM Bobathon: Build for Bengaluru** hackathon.

---

## What it does

CorridorSync combines:

- **Observed traffic** from the TomTom Traffic Flow API (speed, jam factor).
- **Opted-in commuter intentions** from employees across companies.
- **Predicted demand** on road corridors in 15-minute intervals.
- **Departure-time recommendations** based on demand + observed congestion.

> **Data separation guarantee**: observed traffic data and CorridorSync commuter
> demand are always labelled separately and never conflated.  
> Synthetic / demo records are always marked `simulated: true`.

---

## Prerequisites

| Tool | Minimum version |
|---|---|
| Docker | 24 |
| Docker Compose | v2 (bundled with Docker Desktop) |

No other tools need to be installed on the host.

To run backend tests without Docker (e.g. during development):

| Tool | Version |
|---|---|
| Python | 3.12 |
| [uv](https://docs.astral.sh/uv/) | 0.5+ |

---

## Quick start

### 1. Clone the repository

```bash
git clone <repo-url>
cd corridorsync
```

### 2. Create your `.env` file

```bash
cp .env.example .env
```

Open `.env`. The app runs fully in demo mode without any API keys.
If you have a TomTom API key, add it to enable live traffic:

```env
TOMTOM_API_KEY=your_key_here
TRAFFIC_PROVIDER=tomtom
```

Leave `TRAFFIC_PROVIDER=demo` (the default) to use deterministic simulated
traffic data instead. All simulated readings are labelled `simulated: true`.

### 3. Start all services

```bash
docker compose up --build
```

This command:
1. Builds the backend and frontend Docker images.
2. Starts PostgreSQL 15 + PostGIS 3.
3. Runs Alembic migrations (`alembic upgrade head`).
4. Starts the FastAPI backend with hot-reload.
5. Starts the Vite dev server with hot-reload.

First build takes ~2–3 minutes. Subsequent starts are fast.

### 4. Open the application

| Service | URL |
|---|---|
| Frontend (React) | http://localhost:5173 |
| Backend API docs | http://localhost:8000/docs |
| Health check | http://localhost:8000/api/health |

---

## Health check

```bash
curl http://localhost:8000/api/health
```

Expected response when everything is healthy:

```json
{
  "status": "ok",
  "app_name": "CorridorSync",
  "app_env": "development",
  "version": "0.1.0",
  "timestamp": "2026-11-11T07:00:00.000000+00:00",
  "python_version": "3.12.x",
  "database": {
    "connected": true,
    "message": "Database reachable"
  }
}
```

If `database.connected` is `false`, the PostgreSQL container is still starting.
Wait a few seconds and retry.

---

## Running backend tests

### With Docker (recommended for CI)

```bash
docker compose exec backend pytest -v
docker compose exec backend pytest --cov=app --cov-report=term-missing
```

### Without Docker (requires Python 3.12 + uv)

```bash
cd backend
uv run pytest tests/ -v
```

Tests use an in-memory SQLite database — no PostgreSQL required. All 168 tests
pass without any external services or API keys.

---

## Stopping services

```bash
docker compose down          # stop and remove containers
docker compose down -v       # also remove the postgres_data volume (DESTROYS ALL DATA)
```

> ⚠ `docker compose down -v` is destructive. The database volume will be lost.

---

## Environment variables

See [`.env.example`](.env.example) for the full list.

| Variable | Required | Default | Notes |
|---|---|---|---|
| `DATABASE_URL` | Yes | set by Compose | Override only when running outside Docker |
| `TRAFFIC_PROVIDER` | No | `demo` | `tomtom` · `demo` · `none` |
| `TOMTOM_API_KEY` | No | — | Required only when `TRAFFIC_PROVIDER=tomtom` |
| `APP_ENV` | No | `development` | |
| `ORCHESTRATE_INSTANCE_URL` | No | — | Required only to use the Orchestrate agent |
| `ORCHESTRATE_API_KEY` | No | — | Required only to use the Orchestrate agent |

---

## Submitting commuter intentions

```bash
curl -X POST http://localhost:8000/api/v1/commutes/intents \
  -H "Content-Type: application/json" \
  -d '{
    "employee_id": "emp_demo_001",
    "origin_zone_id": 1,
    "destination_zone_id": 3,
    "mode": "car",
    "intended_departure": "2026-11-12T08:30:00+05:30",
    "company_id": "company_a"
  }'
```

---

## watsonx Orchestrate integration

The OpenAPI spec for importing CorridorSync as a set of tools into
watsonx Orchestrate is at [`orchestrate/corridorsync-openapi.yaml`](orchestrate/corridorsync-openapi.yaml).

The agent system prompt is at [`orchestrate/agent-system-prompt.md`](orchestrate/agent-system-prompt.md).

See [`docs/watsonx-orchestrate-integration.md`](docs/watsonx-orchestrate-integration.md)
for full registration instructions.

---

## Project structure

See [`docs/repository-structure.md`](docs/repository-structure.md) for the
full annotated directory tree.

Key directories:

```
backend/app/        FastAPI application
backend/tests/      pytest test suite (168 tests)
frontend/src/       React + TypeScript source
docs/               Architecture and design documents
orchestrate/        watsonx Orchestrate OpenAPI spec + agent prompt
```

---

## Architecture

See [`docs/architecture.md`](docs/architecture.md) for the full component
diagram and design decisions.

---

## Implementation status

| Feature | Status |
|---|---|
| Docker Compose stack (backend + frontend + PostgreSQL) | ✅ Done |
| Alembic DB migrations (zones, corridors, intents, demand, traffic cache) | ✅ Done |
| `GET /api/health` | ✅ Done |
| Commuter profile + daily intent submission | ✅ Done |
| Demand aggregation into 15-minute buckets | ✅ Done |
| `GET /api/v1/demand/forecast` | ✅ Done |
| TomTom Traffic Flow adapter (live + demo fallback) | ✅ Done |
| `GET /api/v1/traffic/segment` | ✅ Done |
| Deterministic departure-recommendation engine | ✅ Done |
| `GET /api/v1/recommend` | ✅ Done |
| React frontend (search → corridor → recommendation) | ✅ Done |
| Demand bar chart (15-min intervals, observed/demand labelled) | ✅ Done |
| Recommendation card with reasoning + alternatives | ✅ Done |
| watsonx Orchestrate OpenAPI spec + agent prompt | ✅ Done |
| Backend test suite — 168 tests, all passing | ✅ Done |

---

## Known limitations (MVP)

- No user authentication.
- No real-time push notifications.
- Commuter demand minimum group size not enforced (privacy improvement for production).
- Continuous GPS tracking is intentionally not implemented.
- Exact home addresses are never stored.
- watsonx Orchestrate integration is configuration-ready; live agent requires
  an active Orchestrate instance and API key.

---

## Privacy

See [`docs/privacy.md`](docs/privacy.md) for the full privacy commitment.

Key points:
- Users select a named **zone**, not a street address.
- No GPS coordinates are stored.
- No continuous location tracking.
- All employee IDs are opaque strings — no PII required.
