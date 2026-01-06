# Stage 1 — Walking Skeleton MVP

## Goal
Produce **1–2 WeChat-ready items per day** from manual inputs (Telegram bot + web upload) with no auto-posting.

## Scope
* Manual ingestion via TG bot / web upload
* Media normalization (resize / transcode)
* Auto-translation & summarisation (zh & en within WeChat caps)
* Simple rule-based selector
* Export writer (+ manifest) to `/content_bank/` folder
* FYI notification to Community Manager via Telegram

## Deliverables
1. Ingestion endpoints: `POST /ingest/telegram`, `POST /ingest/web` (subject & timeframe required).
2. Media processing: images ≤1080 px, videos ≤720 p MP4; SHA-256 checksums; exact dedup.
3. Translation + summary step that respects character caps (CN≤220, EN≤200).
4. Selector that:
   * filters `status = processed`, not expired, dedup-pass
   * picks ≥1 item per day based on recency & timeframe weight
   * supports **force-pick override** (CM can nominate an item to be exported regardless of selector ranking)
5. Export writer that emits caption/media + `manifest.json` under `/content_bank/YYYY-MM-DD/` **with optional `--dry-run` flag** (writes to `/content_bank/_dryrun/` for validation).
6. Telegram FYI message to CM after export.
7. Per-curator **rate limit** (e.g., 30 ingests/day) to avoid spam.

## Success Checks
* System exports on time for **5 consecutive days**.
* CM rates ≥70 % of items as “publishable” (👍) using simple feedback toggle.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Undefined quality leading to bad picks | Enforce strict caption caps & tone template; gather CM feedback early. |
| Media processing failures | Retry logic + fallback to original when transform fails (flagged). |

## Mapping to Project Status Board
* T4 – Media storage service
* T5 – Ingestion API
* T6 – Telegram bot MVP
* T12 – Selector service (baseline)
* T14 – Auto-export writer
* T13.1 – FYI notification

---
### Implementation Tweaks (running log)
- 2025-09-16: Added Alembic integration to API image; migrations now bundled and run in container.
- 2025-09-16: Dropped optional `vector` extension to allow base Postgres image.
- 2025-09-16: Added `fresh_until_ts` column via migration `003_add_fresh_until_ts` to satisfy /ingest/web handler.
- 2025-09-16: Added `provenance` JSON column via migration `004_add_provenance`.
- 2025-09-16: Fixed /ingest/web to store payload as JSON string (asyncpg dict error).
- 2025-09-16: Worker Dockerfile now copies full service code so pipelines (Telegram notify) work.
- 2025-09-16: Telegram bot credentials & super-group chat ID corrected; FYI test message delivered successfully.
- 2025-09-17: JWT header fix for /ingest/telegram; now reads Authorization header via Header().
- 2025-09-17: Store ingest_requests.payload as JSON string to satisfy asyncpg codec.
- 2025-09-17: BAN_REGEX compile hardened; strips inline (?i) to prevent re.error.
- 2025-09-17: Stage-1 end-to-end test passed (ingest → process → selector → dry-run export + Telegram FYI).
- 2025-10-10: Added `/ingest/web/upload:start` presigned upload helper (returns internal media_url and a public presigned URL for host uploads). Bot now uploads TG media to MinIO first and sends media_url in `/ingest/telegram` payload. API `ingest_telegram` records `media_url` and falls back to Telegram download only when needed.
- 2025-10-10: Gateway (`ops/nginx.conf`) updated to proxy `/api/*` to `api:8000` with prefix strip and Docker DNS resolver; removed unused `/worker/healthz` route.
- 2025-10-10: Compose cleanup: removed temporary `api-forwarder` and host port 18080; gateway is the canonical entry point for host clients (`http://127.0.0.1/api`).

*Last updated: 2025-09-17*
