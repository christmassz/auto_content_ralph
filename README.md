# JL Capital – Content Automation Platform

This repository contains the backend services, worker jobs, and infrastructure for the JL Capital content-automation pipeline that prepares daily WeChat-ready content packs.

---

## 📑 Staged Product Roadmap
The product is developed in **eight contractual stages**.  Each stage freezes a stable contract before the next layer is added.  Detailed specifications live in the `Docs/` directory:

| Stage | Document |
| ----- | -------- |
| Stage 0 – Pre-flight & Contracts | [Docs/Stage0_Preflight.md](Docs/Stage0_Preflight.md) |
| Stage 1 – Walking Skeleton MVP | [Docs/Stage1_Walking_Skeleton.md](Docs/Stage1_Walking_Skeleton.md) |
| Stage 2 – Quality & Safety Nets | [Docs/Stage2_Quality_Safety.md](Docs/Stage2_Quality_Safety.md) |
| Stage 3 – Search & RAG Plug-in | [Docs/Stage3_Search_RAG.md](Docs/Stage3_Search_RAG.md) |
| Stage 4 – Website + Librarian Tools | [Docs/Stage4_Website_Librarian.md](Docs/Stage4_Website_Librarian.md) |
| Stage 5 – Ops Hardening & Observability | [Docs/Stage5_Ops_Hardening.md](Docs/Stage5_Ops_Hardening.md) |
| Stage 6 – Content Intelligence | [Docs/Stage6_Content_Intelligence.md](Docs/Stage6_Content_Intelligence.md) |
| Stage 7 – Autopost & Multichannel | [Docs/Stage7_Autopost_Multichannel.md](Docs/Stage7_Autopost_Multichannel.md) |

> The **contracts in Stage 0 are immutable**; later stages may extend but never break them.

---

## 🐳 Quick Start (Local)
```bash
# 1. Copy environment template
cp env.template .env
# 2. Fill in secrets in .env (DB, MinIO, JWT, OpenAI…)
# 3. Boot the stack
docker compose up --build
# 4. Verify health endpoints
curl http://localhost:8000/healthz

# Stage-2 env tweaks
export SKIP_TODAY=false            # set true to pause daily export
export EXCLUDE_IDS=""             # comma-separated content UUIDs to hide

# New Python deps
pip install imagehash fakeredis
```

### Common Services
| Service | Port | Notes |
| ------- | ---- | ----- |
| API (FastAPI) | 8000 | Core HTTP API & auth |
| Worker | – | Background jobs (Redis queue) |
| Postgres + pgvector | 5432 | Data store |
| Redis | 6379 | Job queue |
| MinIO (S3) | 9000 / 9001 | Media storage + console |
| RAG-Anything | 8001 | Vector search |
| Nginx gateway | 80 | Reverse proxy |

---

## 📂 Repo Layout
```
services/      # Dockerised micro-services (api, worker, rag)
ops/           # Infra scripts (db init, nginx conf, minio bootstrap)
Docs/          # Stage specifications (immutable contracts)
scripts/       # One-off helper scripts (e.g., issue_token.py)
```

---

## 🧪 Tests & CI
Minimal CI (Python 3.11) runs lint and unit tests on every PR.  See `.github/workflows/ci.yml`.

---

## 🤝 Contributing
Please read and **agree not to break Stage 0 contracts** before submitting a PR.  Each change should reference a *Stage* and *Task* ID from `.cursor/scratchpad.md`.

---

*Last updated 2025-09-18*
