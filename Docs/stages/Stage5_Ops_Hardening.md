# Stage 5 — Ops Hardening & Observability

## Goal
Make day-to-day operations **boring and reliable**.

## Scope
* CI/CD, metrics, alerts, backups
* Security basics (rate limits, secret rotation)
* Runbooks & restore drills
* Cost & resource monitoring (API usage, disk quotas)
* Separate **staging vs production** deployment profiles

## Deliverables
1. **CI pipeline**: lint, unit tests, integration ingest→export, schema-drift check.
2. **Metrics & alerts**: ingest/day, dedup rate, selector runtime; alert if selector not done by 09:15 local. **Cost metrics**: embeddings API $, translation $, transcode CPU h; alert budgets (e.g., >$10/day).
3. **Backups**: DB, MinIO buckets, RAG index nightly; monthly restore drill.
4. **Security**: token TTL/rotation, per-IP rate limits, input caps, **PII detection hooks (disabled by default)**.
### Host-run bot and gateway routing (2025-10-10)

- To survive flaky Docker egress to Telegram, we run the Telegram bot on the host with a systemd user unit `content-bot.service` and point `API_BASE` at the gateway (`http://127.0.0.1/api`).
- Gateway config strips `/api` prefix and uses Docker DNS resolver so API restarts don’t cause 502s.
- A systemd user unit `content-stack.service` brings up the core Docker services on login.
- Compose’s temporary `api-forwarder` was removed to avoid port conflicts; gateway is the single entry point.

5. **Disk space quotas** on MinIO volumes & Postgres; alerts at 80 % utilisation.
6. **Schema drift detection** in CI (`alembic check`) preventing accidental un-applied migrations.
7. **Staging environment** definition (separate buckets, DB, index) with auto-deploy on `main`.

### Content backfill enhancements
- PDF text extractor in worker (pdfminer) behind `FEATURE_PDF=true`.
- Chart digitizer → structured series from images; store CSV/JSON alongside items.
- Backfill task: iterate historical items, enrich text/series from preserved media.
- RAG reindex plan: bump `RAG_INDEX_VERSION`, reindex, delete old version, compact.

## Success Checks
* CI green on main; failing builds block merge.
* Alerting tested (dummy alarms) and verified.
* One full restore drill completed successfully.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Missing metrics leading to blind spots | Define SLOs early; dashboards in Grafana. |
| Backup corruption | Automated nightly restore-verify job. |

## Mapping to Project Status Board
* T18 – Observability & ops tasks
* T20 – Rollout plan & docs additions

### RSS/link-fetcher ops scaling

Add proxy pool configuration and a per-domain politeness table (min delay, max RPM). Track metrics for fetch success rate, 4xx/5xx per domain, fetch latency, and captcha occurrence. Introduce a persistent queue (Redis streams or SQS) for fetch jobs and autoscale workers. Alert on repeated failures for specific domains.

---
*Last updated: 2025-09-09*
