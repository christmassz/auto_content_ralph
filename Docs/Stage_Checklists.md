# Stage Progress Checklists

This file provides a concise checkbox list for each stage so the team can track progress at a glance.  Tick items off in PR descriptions or commit messages as you complete them.

---

## Stage 0 — Pre-flight & Contracts ✅
- [x] Contracts package (`jl-contracts`) published
- [x] Manifest v1 JSON Schema frozen
- [x] `schema_version` & `manifest_version` fields added
- [x] Feature flags wired (`FEATURE_*`)
- [x] Alembic baseline migration (`000_initial`) applied
- [x] Health endpoints live (`/healthz`, `/healthz/db`, `/healthz/storage`)
- [x] CI lint + unit + schema drift in place

## Stage 1 — Walking Skeleton MVP
- [x] `POST /ingest/telegram` endpoint
- [x] `POST /ingest/web` endpoint
- [x] Media normalization (≤1080 px img, ≤720p video)
- [x] Exact SHA-256 dedup
- [x] Auto-translation & summary within CN≤220 / EN≤200 chars
- [x] Rule-based selector (recency/timeframe)
- [x] Force-pick override & per-curator rate limit
- [x] Export writer to `/content_bank/YYYY-MM-DD/` at run time
- [x] Telegram FYI after export
- [x] Dry-run export path (`_dryrun/`)

## Stage 2 — Quality & Safety Nets
- [x] Alembic migration: add columns `similarity_to_recent`, `phash`, `freshness_score`, `quality_proxy`, `selector_score`, `is_unverified`
- [x] Update SQLAlchemy models & Pydantic schemas
- [x] SimHash (text) near-dup detection
- [x] pHash (image) near-dup detection
- [x] `similarity_to_recent` column populated
- [x] Scoring function (`freshness`, `quality_proxy`)
- [x] `selector_score` integrated
- [x] Limit ≤1 `unverified` item/day (hedged caption)
- [x] `.skip_today` & `exclude_ids` overrides
- [x] Per-curator Redis token bucket enforced via Redis
- [x] Unit tests for dedup, scoring, throttle
- [x] Docs & README updated for Stage 2

## Stage 3 — Search & RAG Plug-in
- [x] RAG-Anything service containerised
- [x] `/search` proxies to RAG when `FEATURE_RAG=true`
- [x] Unified worker-only OCR (tessdata_best, PSM=6, original image, tweet-tuned)
- [x] Worker-triggered RAG indexing on `content.updated` (chunking + batched `/upsert`)
- [x] Automatic fallback to Postgres search on RAG failure
- [x] Selector novelty penalty using RAG cosine similarity (fold into selection)
- [x] Idempotent RAG upsert (respect `chunk_id`; avoid dupes)
- [x] RAG result ID semantics aligned (return `content_id` for API expansion)
- [x] RAG client honors `RAG_QUERY_TIMEOUT` env
- [x] Versioned index semantics honored (`rag_index_version` exposed; active version persisted)
- [x] RAG management endpoints: delete/tombstone, compact, rebuild
- [x] Adapter base interface and registry in `services/worker/adapters/`
- [x] SubstackAdapter (no fetch): parse provided title/snippet/meta
- [x] TwitterAdapter: pass-through tweet_text/caption
- [x] GenericAdapter: excerpt-only fallback
- [x] Worker integration before summarize/score; update `raw_text` and provenance
- [x] API allow-list helper (permissive by default; strict behind flag)
- [x] Privacy defaults set: `FEATURE_USAGE_METRICS=false`, `LOG_PROMPTS=false`, `SCRUB_PII=true`
- [x] Ports doc clarified: host 8081 ↔ container 8088; env aligned
- [x] `/search?expand=true` enrichment implemented
- [x] Cosine-normalized vectors in RAG; richer `/healthz`
- [x] Acceptance script `scripts/check_stage3.sh` added and green
- [x] `RAG_URL=http://rag:8088` (in-cluster)
- [x] RAG host ports removed from compose; health checked from API container
- [x] Docs updated to prefer in-cluster checks; host 8081 optional only

### Stage 3 — Granular implementation plan (pending items)
- [x] Worker → RAG upsert: use shared chunk builder with `/admin/reindex`, retry/backoff
- [x] `/search` fallback: catch RAG errors, route to rule-based/DB and set `rag_used=false`
- [x] `RagClient`: use env `RAG_QUERY_TIMEOUT`; map results to `{ id: content_id, score }`
- [x] RAG upsert idempotency: `IndexIDMap2` + 64-bit hash of `chunk_id` as FAISS IDs
- [x] RAG maintenance: `/delete`, `/compact`, `/rebuild` (by `rag_index_version`)
- [ ] Optional read-after-write probe for upsert batches
- [x] Selector novelty: compute cosine vs last N items via RAG; fold with weights
- [x] Docs: update Stage3 guide and User Manual with exact API/env snippets

## Stage 3.5 — Content Ingestion QoL
- [x] File-based RSS configuration (`ops/rss_feeds.txt` + `RSS_FEEDS_FILE`)
- [x] Daily RSS poll with idempotent ingest
- [x] Telegram librarian flows (DM/group)
- [x] Web ingest API documented and usable
- [ ] Optional: websites.txt + simple web ingester (future)
- [ ] Operator docs for quick add/verify of sources

## Stage 4 — Website Search & Librarian Tools
- [ ] `/content` search page with filters/pagination
- [ ] Detail edit view (tags, timeframe, confidence, note)
- [ ] Merge-duplicates workflow & API
- [ ] Append-only audit log surfaced in UI
- [ ] Candidate-aware retrieval: intersect RAG results with DB candidates (or prefilter)
- [ ] Query-time filter by `rag_index_version` in RAG `/query`
- [ ] Define/implement `mode` (semantic|hybrid|exact) or remove

## Stage 5 — Ops Hardening & Observability
- [ ] Cost metrics (embeddings $, disk)
- [ ] Disk quota alerts (MinIO, Postgres)
- [ ] Nightly backups (DB, MinIO, RAG)
- [ ] Monthly restore drill doc + script
- [ ] CI staging vs production profiles
- [ ] Schema drift check gates merge
- [ ] PII detection hooks scaffolded (flag OFF)
- [ ] Token revocation endpoint + blacklist table
- [ ] Align/remove unused PG `embedding` column/index (FAISS is primary)
- [ ] Add API `depends_on: rag` (optional) for startup ordering
- [ ] Fix RagClient test to patch `httpx.Client.post`

## Stage 6 — Content Intelligence
- [ ] TG feedback buttons wired (👍 / meh / skip)
- [ ] Feedback stored & exposed via API
- [ ] Scoring v2 (learned weights or small model)
- [ ] Selector A/B toggle
- [ ] “Aged-well” bias implemented

## Stage 7 — Autopost & Multichannel
- [ ] Auto-post module to WeChat with retry/logging
- [ ] Dry-run + pause switch for autopost
- [ ] Channel templates for TG Channel & X
- [ ] Rollback procedure documented
