# Stage 0 — Pre-flight & Contracts

## Goal
Freeze the core interfaces and schemas so that future features (RAG, near-dup, RBAC, etc.) can plug in without rewrites.

## Scope
* Database schemas
* Status / enum definitions
* Manifest v1 contract
* Health check endpoints
* Feature flags
* Baseline CI workflow
* Data retention & governance baseline
* Audit infrastructure stub

## Deliverables
1. PostgreSQL tables `content_items` and `media_assets` with future-proof columns (`quality_score`, `novelty_score`, `rag_index_version`, etc.).
2. Stable **Manifest v1** (additive changes only after this point) **with `manifest_version: 1` field**.
3. Both tables include a `schema_version` column (INT DEFAULT 0) so downstream consumers can detect migrations.
4. Retention defaults established:
   * Originals kept **90 days** (unless `evergreen`), normalized derivatives **365 days**, manifests retained **forever**.
   * Policy encoded in config and enforced by Stage 5 cleanup job.
5. Public endpoints: `/healthz`, `/healthz/db`, `/healthz/storage` all return **200 OK**.
6. Feature flags created and **defaulted to false**:
   * `FEATURE_RAG`
   * `FEATURE_NEAR_DUP`
   * `FEATURE_CM_FEEDBACK`
   * `EXPORT_AUTOPOST`
7. Audit log table stub (`audit_events`) capturing **all ingest / edit / export actions** (even if only populated later).
8. Token management baseline: endpoints and table to support **immediate token revocation** (blacklist) in addition to TTL/rotation.
9. Minimal CI pipeline that runs lint + unit tests on every push / PR, **plus schema-drift check** (`alembic upgrade --sql`).

## Success Checks
* `docker-compose up` starts all containers successfully.
* A sample ingest → export round-trip executes locally without errors.
* All `/healthz*` endpoints return green status.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Schema creep / breaking changes later | Lock enums & statuses now; only additive migrations allowed after Stage 0. |
| Incomplete health coverage | Add smoke tests in CI to hit every `/healthz*` route. |

## Mapping to Project Status Board
* T1 – Repo & environment scaffolding
* T2 – Database schema & migrations
* S1–S6 – Immediate setup tasks
* T18.1 – Health endpoints

---
*Last updated: 2025-09-09*
