# CorridorSync — Agent Instructions

CorridorSync is a crowd-powered commute intelligence platform for Bengaluru,
built for the **IBM Bobathon: Build for Bengaluru** hackathon.

---

## Project Objective

CorridorSync combines:

1. Live traffic-flow information from external providers.
2. Opt-in daily commuter intentions from employees across multiple companies.
3. Predicted demand on road corridors in 15-minute intervals.
4. Route and departure-time recommendations.

---

## Technology Stack

| Layer          | Technology                                      |
| -------------- | ----------------------------------------------- |
| **Frontend**   | React, TypeScript, Vite, Tailwind CSS, Leaflet  |
| **Backend**    | Python, FastAPI, Pydantic                       |
| **Database**   | PostgreSQL with PostGIS                         |
| **Infrastructure** | Docker Compose                              |
| **IBM tooling** | IBM Bob (development), IBM watsonx Orchestrate (runtime agent coordination) |

---

## Project Constraints

1. **Scope** — Keep the project suitable for one developer; build an MVP, not an enterprise-scale application.
2. **Deadline** — Target submission is **November 14, 2026**.
3. **Data separation** — Always keep observed (external) traffic data separate from predicted CorridorSync commuter demand. Never conflate the two.
4. **No fabricated speeds** — Never convert commuter counts directly into actual traffic speeds.
5. **No invented data** — Do not invent live traffic data or route ETAs.
6. **Synthetic records** — Label any synthetic or mock records clearly (e.g. `synthetic: true` field or a `[SYNTHETIC]` prefix in descriptions).
7. **Privacy** — Never store exact home addresses. Do not implement continuous GPS tracking.
8. **Secrets** — Keep all external API keys out of source code; use environment variables or a secrets manager.
9. **Determinism** — Use deterministic Python functions for all numerical calculations and recommendations.
10. **watsonx Orchestrate role** — Use watsonx Orchestrate for agent coordination only. Do not use it as a replacement for backend calculations.

---

## Development Rules

### General

- Work on **one feature at a time**. Do not implement application functionality unless explicitly asked.
- Do not modify files unrelated to the current task.
- Explain every file you create or change — what it is, what it does, and why it is needed.

### Code quality

- Follow existing code style and project conventions in every file you touch.
- Produce the minimal change that solves the problem; do not add features, refactors, or abstractions beyond what was asked.
- Use meaningful, descriptive names for variables, functions, and endpoints.

### Testing

- Add tests for every important backend feature.
- Run tests before declaring a feature complete.
- Use `pytest` for Python backend tests.

### Git

- Use meaningful Git commit messages that describe *what* changed and *why*.
- One logical change per commit; do not bundle unrelated changes.

### Infrastructure

- Do not install unnecessary infrastructure or third-party dependencies.
- Ask for approval before running any destructive command (dropping tables, deleting data, removing containers, etc.).

### Secrets & environment

- All API keys, database credentials, and secrets must live in `.env` files or environment variables.
- Never hard-code credentials anywhere in source code or committed configuration files.
- Add `.env` to `.gitignore` from the start.

---

## Repository Layout (planned)

```
corridorsync/
├── backend/          # FastAPI application
│   ├── app/
│   │   ├── api/      # Route handlers
│   │   ├── models/   # Pydantic & SQLAlchemy models
│   │   ├── services/ # Business logic (deterministic calculations)
│   │   └── db/       # Database setup and migrations
│   └── tests/        # pytest test suite
├── frontend/         # React + Vite + Tailwind application
│   └── src/
├── infra/            # Docker Compose and infrastructure config
├── .env.example      # Template for required environment variables
├── AGENTS.md         # This file
└── README.md         # Project overview and setup guide
```

---

## Key Domain Concepts

| Term | Definition |
| ---- | ---------- |
| **Corridor** | A named road segment or route in Bengaluru (e.g. "Outer Ring Road — Marathahalli to Silk Board"). |
| **Commuter intention** | An opt-in, anonymised record of when and where an employee plans to commute, submitted each morning. |
| **Demand interval** | A 15-minute time bucket used to aggregate predicted commuter demand on a corridor. |
| **Observed traffic** | Real-world speed/flow data sourced from an external provider (e.g. HERE, TomTom). |
| **Predicted demand** | CorridorSync's own aggregation of commuter intentions onto corridors — never presented as a traffic speed. |
| **Recommendation** | A suggested route and/or departure-time window derived from predicted demand and observed traffic, produced by a deterministic backend function. |

---

## watsonx Orchestrate Integration Notes

- watsonx Orchestrate acts as the **agent coordinator**: it receives user queries, calls the appropriate CorridorSync backend API tools, and assembles a natural-language response.
- All heavy computation (demand aggregation, recommendation scoring) happens in the **FastAPI backend**, not inside the Orchestrate agent.
- The Orchestrate agent must never fabricate traffic data or commuter counts; it must always surface values returned by the backend.

---

## Out of Scope for MVP

- Real-time push notifications.
- Mobile native application.
- Multi-city support.
- Continuous or background GPS collection.
- Machine-learning model training (use deterministic heuristics only).
