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
*Last updated: 2025-09-09*
