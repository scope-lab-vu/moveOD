# MoveOD: Synthesizing Fine-Grained Origin-Destination Data for Transportation CPS

MoveOD is a framework for synthesizing fine-grained origin-destination (OD) demand data for transportation cyber-physical systems. It combines GTFS-based fixed-line transit planning, on-demand mobility services, and OD demand estimation into a full-stack application with a FastAPI backend and React frontend.

The system integrates multiple data sources -- LODES (employment), INRIX (traffic), OpenStreetMap (buildings/POIs), and Census geographies -- and applies integer linear programming (ILP) calibration to produce building-level OD matrices suitable for digital twin simulations and transit planning.

## Prerequisites

- Python 3.12+
- Node.js 18+
- PostgreSQL with PostGIS extension
- [uv](https://docs.astral.sh/uv/) (Python package manager)
- Docker and Docker Compose (for database infrastructure)
- OSRM (optional, for routing)

## Quick Start

### 1. Environment Configuration

Copy the example environment file and adjust values as needed:

```bash
cp .env.example .env
```

See `.env.example` for all available configuration options including database credentials, API settings, planning defaults, and routing configuration.

### 2. Docker (Postgres + PostGIS + pgAdmin + optional OSRM)

```bash
cd docker
docker compose --env-file ../.env up -d
```

To stop services:

```bash
docker compose --env-file ../.env down
```

Notes:
- Postgres data persists in the `postgres_data` volume.
- pgAdmin runs at `http://localhost:${PGADMIN_PORT}`.
- OSRM is optional. Set `COMPOSE_PROFILES=osrm` in `.env` to enable.

### 3. Backend

```bash
cd backend

uv sync
uv run python scripts/init_db.py
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Generate OpenAPI.json:

```bash
uv run python scripts/generate_openapi.py
```

Load GTFS:

```bash
uv run python scripts/load_gtfs.py --path ./scripts/data/gtfs/MERGED_gtfs.zip
```

Load on-demand fleet (optional):

```bash
uv run python scripts/load_ondemand.py
```

MoveOD utilities:

```bash
uv run python scripts/init_db.py --drop-analysis
uv run python scripts/load_moveod.py
# Optional (should be replaced by actual generation)
uv run python scripts/load_moveod_demand.py
```

### 4. Frontend

```bash
cd frontend

npm install
npm run generate
npm run dev
```

## Tests

Tests use a separate database defined by `DATABASE_URL_TEST`. See `.env.example`.

```bash
cd backend
uv run pytest
```

## Project Structure

```
moveOD/
├── backend/            # FastAPI REST API
│   ├── app/
│   │   ├── api/        # Route handlers
│   │   ├── models/     # SQLAlchemy + PostGIS models
│   │   ├── schemas/    # Pydantic request/response schemas
│   │   ├── services/   # Planning, routing, and analysis logic
│   │   ├── moveod/     # OD demand synthesis (ILP calibration, data processing)
│   │   └── crud/       # Database access layer
│   ├── scripts/        # Data loading and utility scripts
│   └── tests/          # pytest test suite
├── frontend/           # React + TypeScript UI
│   └── src/
│       ├── components/ # Map views, planning UI, MoveOD analysis pages
│       └── services/   # Auto-generated API client (Zodios)
└── docker/             # Docker Compose (PostGIS, pgAdmin, OSRM)
```

## Citing This Work

If you use MoveOD in your research, please cite:

```bibtex
@inproceedings{iccps2026_moveod,
  author    = {Sen, Rishav and Talusan, Jose Paolo and Dubey, Abhishek and Mukhopadhyay, Ayan and Samaranayake, Samitha and Laszka, Aron},
  title     = {MoveOD: Synthesizing Fine-Grained Origin--Destination Data for Transportation CPS},
  year      = {2026},
  booktitle = {Proceedings of the HSCC/ICCPS 2026: 29th ACM International Conference on Hybrid Systems: Computation and Control and 17th ACM/IEEE International Conference on Cyber-Physical Systems},
  location  = {Saint Malo, France},
  keywords  = {origin-destination data, transportation systems, data synthesis, digital twins},
  note      = {Acceptance rate: 28\%; Short Paper; Track: Systems and Applications},
  series    = {HSCC/ICCPS '26}
}
```

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to share and adapt this material for any purpose, provided you give appropriate credit, provide a link to the license, and indicate if changes were made.
