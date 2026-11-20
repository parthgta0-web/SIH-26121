# NAMOWELL AI

NAMOWELL AI is a Smart India Hackathon team prototype for **Oil India's Nearby Wells Intelligence System (problem statement 26121)**. It gives reviewers a way to find relevant nearby wells, compare historical incidents by depth and formation, and trace extracted information back to the source report.

The app runs locally with a demonstration database. Its relevance scores and alert states help explain the demo workflow. They are unsuitable for operational drilling decisions.

## My contributions — Paarth Gupta

I led the technical work and built the core system, from the data model and backend services to the dashboard workflows.

- **Nearby-well ranking.** Built a deterministic scoring engine that combines geographic distance, formation match, depth overlap, trajectory, and event relevance. The API returns the score breakdown and missing factors so reviewers can see why a well ranks where it does.
- **Depth correlation and alerting.** Built the workflow that compares replayed well depth against incident intervals in nearby wells. The app shows when an interval is upcoming, in range, or passed and links the alert to the supporting records.
- **Report intake and review.** Implemented local PDF and TXT extraction, optional OCR for scanned PDFs, duplicate-file detection, and a review queue for candidate incidents. Each candidate retains a source page or line and a review status.
- **API, dashboard, and evaluation.** Built the FastAPI and SQLAlchemy backend and connected it to a React dashboard with a Leaflet map, depth-correlation charts, incident filters, and replay controls. Added a benchmark view to examine alert precision, recall, lead distance, and processing latency on demo scenarios.

## Try the demo

1. Choose an active-well scenario on the dashboard. Open an offset-well row to inspect the score factors, missing fields, and provenance.
2. Step through depth in the simulation controls. Compare the alert state and the corresponding historical interval in the depth chart.
3. Open the incident explorer to filter by event type, severity, or formation and inspect the associated report evidence.
4. Upload a sample PDF or TXT file in **Report Intake**. Review the extracted candidates and their source references.
5. Open the benchmark view to replay a demo scenario and inspect its metrics.

## How the ranking works

The [offset engine](backend/app/services/offset_engine.py) applies a distance cutoff and scores the remaining candidates. These are the default demo weights:

| Factor | Weight |
| --- | ---: |
| Geographic proximity | 25% |
| Formation match | 30% |
| Depth overlap | 20% |
| Trajectory match | 10% |
| Event relevance | 15% |

Distance uses a linear decay within the search radius. Missing factors are excluded and the weights of the available factors are rescaled. The final score ranges from 0 to 100 and represents relative relevance within the prototype.

## Run locally

From the repository root, start the backend in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r backend/requirements.txt
python scripts/seed_database.py
python -m uvicorn backend.app.main:app --reload --port 8000
```

In a second terminal, from the repository root:

```powershell
cd frontend
npm ci
npm run dev
```

Open the [dashboard](http://localhost:5173) and the [interactive API docs](http://localhost:8000/docs). The frontend uses `http://localhost:8000` by default. Set `VITE_API_URL` in `frontend/.env` to use a different API address. OCR for scanned PDFs requires the Tesseract executable.

## Code map

| Path | What to look for |
| --- | --- |
| [`backend/app/services/`](backend/app/services) | Offset ranking, depth correlation, risk rules, and replay benchmarking |
| [`backend/app/api/`](backend/app/api) | API routes for wells, incidents, telemetry, risk, and documents |
| [`frontend/src/`](frontend/src) | React and TypeScript dashboards, Leaflet mapping, and Recharts visualizations |
| [`data/`](data) | Demo fixtures and the dataset dictionary |
| [`tests/`](tests) | API and domain tests for ranking, alerts, document handling, and data adapters |

Run the backend tests from the repository root with `python -m pytest`. Run the frontend tests from `frontend/` with `npm test`.

## Data and evaluation notes

- The default setup seeds generated scenarios from `data/synthetic/`. Some Volve-themed fixtures carry `is_synthetic: false` despite being generated. Treat their coordinate, incident, and report values as demo data.
- [The optional Volve data notes](data/real/README.md) describe a separate public dataset workflow. The seeded demo uses no Oil India operational records.
- Benchmark results reflect the replayed demo events. The experimental classifier reports a score on generated training data. It has no held-out field validation.
- The prototype uses local SQLite storage and optional map tiles. Production eRTMAC integration and operational validation remain outside the demo.

## Documentation and credits

- [Architecture notes](docs/architecture.md)
- [Technical report source](docs/WellSage_NWIS_Technical_Report.tex)
- [Data dictionary](data/data_dictionary.md)

Built as a team project for Smart India Hackathon. This fork preserves the team history from the [original SIH-26121 repository](https://github.com/rajkeshavmi4/SIH-26121).
