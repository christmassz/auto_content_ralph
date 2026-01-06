# Stage Progress Checklists

This file provides a concise checkbox list for each stage so the team can track progress at a glance.  Tick items off in PR descriptions or commit messages as you complete them.

---

## Stage 0 — Pre-flight & Contracts
- [ ] Contracts package (`jl-contracts`) published
- [ ] Manifest v1 JSON Schema frozen
- [ ] `schema_version` & `manifest_version` fields added
- [ ] Feature flags wired (`FEATURE_*`)
- [ ] Alembic baseline migration (`000_initial`) applied
- [ ] Health endpoints live (`/healthz`, `/healthz/db`, `/healthz/storage`)
- [ ] CI lint + unit + schema drift in place

## Stage 1 — Walking Skeleton MVP
- [ ] `POST /ingest/telegram` endpoint
- [ ] `POST /ingest/web` endpoint
- [ ] Media normalization (≤1080 px img, ≤720p video)
- [ ] Exact SHA-256 dedup
- [ ] Auto-translation & summary within CN≤220 / EN≤200 chars
- [ ] Rule-based selector (recency/timeframe)
- [ ] Force-pick override & per-curator rate limit
- [ ] Export writer to `/content_bank/YYYY-MM-DD/` at run time
- [ ] Telegram FYI after export
- [ ] Dry-run export path (`_dryrun/`)

## Stage 2 — Quality & Safety Nets
- [ ] Alembic migration: add columns `similarity_to_recent`, `phash`, `freshness_score`, `quality_proxy`, `selector_score`, `is_unverified`
- [ ] Update SQLAlchemy models & Pydantic schemas
- [ ] SimHash (text) near-dup detection
- [ ] pHash (image) near-dup detection
- [ ] `similarity_to_recent` column populated
- [ ] Scoring function (`freshness`, `quality_proxy`)
- [ ] `selector_score` integrated
- [ ] Limit ≤1 `unverified` item/day (hedged caption)
- [ ] `.skip_today` & `exclude_ids` overrides
- [ ] Per-curator Redis token bucket enforced via Redis
- [ ] Unit tests for dedup, scoring, throttle
- [ ] Docs & README updated for Stage 2

## Stage 3 — Search & RAG Plug-in
- [ ] RAG-Anything service containerised
- [ ] `/search` proxies to RAG when `FEATURE_RAG=true`
- [ ] Unified worker-only OCR (tessdata_best, PSM=6, original image, tweet-tuned)
- [ ] Worker-triggered RAG indexing on `content.updated` (chunking + batched `/upsert`)
- [ ] Automatic fallback to Postgres search on RAG failure
- [ ] Selector novelty penalty using RAG cosine similarity (fold into selection)
- [ ] Idempotent RAG upsert (respect `chunk_id`; avoid dupes)
- [ ] RAG result ID semantics aligned (return `content_id` for API expansion)
- [ ] RAG client honors `RAG_QUERY_TIMEOUT` env
- [ ] Versioned index semantics honored (`rag_index_version` exposed; active version persisted)
- [ ] RAG management endpoints: delete/tombstone, compact, rebuild
- [ ] Adapter base interface and registry in `services/worker/adapters/`
- [ ] SubstackAdapter (no fetch): parse provided title/snippet/meta
- [ ] TwitterAdapter: pass-through tweet_text/caption
- [ ] GenericAdapter: excerpt-only fallback
- [ ] Worker integration before summarize/score; update `raw_text` and provenance
- [ ] API allow-list helper (permissive by default; strict behind flag)
- [ ] Privacy defaults set: `FEATURE_USAGE_METRICS=false`, `LOG_PROMPTS=false`, `SCRUB_PII=true`
- [ ] Ports doc clarified: host 8081 ↔ container 8088; env aligned
- [ ] `/search?expand=true` enrichment implemented
- [ ] Cosine-normalized vectors in RAG; richer `/healthz`
- [ ] Acceptance script `scripts/check_stage3.sh` added and green
- [ ] `RAG_URL=http://rag:8088` (in-cluster)
- [ ] RAG host ports removed from compose; health checked from API container
- [ ] Docs updated to prefer in-cluster checks; host 8081 optional only

### Stage 3 — Granular implementation plan (pending items)
- [ ] Worker → RAG upsert: use shared chunk builder with `/admin/reindex`, retry/backoff
- [ ] `/search` fallback: catch RAG errors, route to rule-based/DB and set `rag_used=false`
- [ ] `RagClient`: use env `RAG_QUERY_TIMEOUT`; map results to `{ id: content_id, score }`
- [ ] RAG upsert idempotency: `IndexIDMap2` + 64-bit hash of `chunk_id` as FAISS IDs
- [ ] RAG maintenance: `/delete`, `/compact`, `/rebuild` (by `rag_index_version`)
- [ ] Optional read-after-write probe for upsert batches
- [ ] Selector novelty: compute cosine vs last N items via RAG; fold with weights
- [ ] Docs: update Stage3 guide and User Manual with exact API/env snippets

## Stage 3.5 — Content Ingestion QoL
- [ ] File-based RSS configuration (`ops/rss_feeds.txt` + `RSS_FEEDS_FILE`)
- [ ] Daily RSS poll with idempotent ingest
- [ ] Telegram librarian flows (DM/group)
- [ ] Web ingest API documented and usable
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
