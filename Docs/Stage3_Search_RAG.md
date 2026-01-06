# Stage 3 — Search & RAG Plug-in ✅

## Goal
Introduce smart retrieval & novelty scoring **without breaking** Stage 1/2 flows.

## Scope
* Stand-alone RAG-Anything service (FAISS index)
* Worker that indexes on `content.updated` events
* `/search` API proxy that toggles via `FEATURE_RAG`
* Selector novelty boost via `IRetrieval.rank`
* Worker-only OCR (unified across sources)

## Current status: built vs pending

### Built
1. **RAG service** with `/upsert`, `/query`, cosine-normalized search, and richer `/healthz` (version, model, uptime); index persisted under `rag_index/`.
2. Admin backfill `POST /admin/reindex` that normalizes text, chunks (≈900/150), enriches metadata (`chunk_id`, `rag_index_version`, `content_id`), batches with retries, and upserts to RAG.
3. API endpoint `GET /search` proxies to RAG when `FEATURE_RAG=true`; `expand=true` enriches results from DB.
4. Worker-only OCR finalized; adapters (Substack/Twitter/Generic) integrated pre-summarization.
5. Worker-triggered RAG indexing on content updates (chunking + batched `/upsert`).
6. Fallback to DB/rule-based when RAG errors; `rag_used=false` returned.
7. Idempotent upsert via FAISS `IndexIDMap2` with stable 63-bit IDs from `chunk_id`.
8. Maintenance endpoints: `/delete`, `/compact`, `/rebuild`; active index version persisted and exposed.
9. Bridged-only networking; in-cluster `RAG_URL=http://rag:8088`. API disables proxy inheritance for in-cluster RAG calls.

### Pending (polish)
1. Optional query-time filter by `rag_index_version` (currently active version reported via `/healthz`).
2. CI job to run `scripts/check_stage3.sh` in compose test profile.
3. Remove compose `version:` warning.

## Success checks (target)
* `/search` returns relevant results and `rag_used=true` when RAG answers; returns `rag_used=false` on fallback with non-error 200.
* Worker logs: `rag_upsert_ms=<n> chunks=<m>` per processed content.
* RAG `/healthz` shows increasing `items` and correct `rag_index_version`.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Re-index storms during model/param change | Use `rag_index_version`; build side-by-side then flip pointer. |
| Increased infra complexity | Keep RAG as isolated service with clear health checks & backups. |
| RAG downtime causing search 500s | Automatic fallback to Postgres search path; alert triggered. |

---

## Implementation plan (granular)

1) Worker → RAG upsert
- Reuse chunking logic from `/admin/reindex` in `services/worker/worker.py` after scoring.
- Batch size: 64; JSON payload cap: 512KB; retry with backoff on 429/5xx (≤3 attempts).
- Attach metadata: `content_id`, `embedding_model`, `rag_index_version`, `chunk_index`, `chunk_id` (SHA256 of `content_id:idx:version`).
- Feature-gate with `FEATURE_RAG`.

2) Idempotent upsert & maintenance (RAG)
- Switch to `faiss.IndexIDMap2(IndexFlatIP)`; compute 64-bit IDs from `chunk_id`.
- Implement `/delete` (by `content_id` or `rag_index_version`) and `/compact` to drop tombstoned entries.
- Persist a sidecar `ids.bin` or encode IDs directly via `IndexIDMap2` to avoid dupes.

3) Query/result semantics
- In `/query`, return `content_id` and `chunk_id`; include `rag_index_version`.
- In `RagClient`, map results to `{"id": content_id, "score": score}`; keep `doc_id` only as auxiliary field if needed.

4) Search fallback & timeouts
- In `/search`, wrap retrieval in try/except; on error, call rule-based/DB path and return `rag_used=false`.
- In `RagClient`, use `RAG_QUERY_TIMEOUT` from settings; keep `RAG_UPSERT_TIMEOUT` in indexers.

5) Versioning & rebuild
- Support `RAG_INDEX_VERSION` env; reindex under a new version; swap by updating env and reloading RAG.
- Implement `/rebuild?version=V` to clear and rebuild (admin-only).

6) Selector novelty via RAG
- For last N processed (e.g., 50), embed titles or summaries and compute cosine via `/query` multi-lookup or direct.
- Fold into `selector_score` with `NOVELTY_WEIGHT` and `SIM_THRESHOLD`.

7) Docs & acceptance
- Add `scripts/check_stage3.sh` to run ingest → worker → RAG upsert → search checks.
- Update User Manual to reflect fallback semantics and new envs.

## Mapping to Project Status Board
* T8.6.2 – Worker task & API search endpoint
* T18.2 – Nightly backups (include RAG index)
* Weekly scaffold: PDF text via pdfminer (live), image OCR reuse, regex family tagging

### Domain adapters (Stage 3 scope)

Implement per-domain adapters to normalize content from Substack/Twitter/Generic sites. Each adapter returns `{title, text_html, text_plain, images[], canonical_url}` and runs within the worker enrichment step. Adapters should be idempotent and tested with representative fixtures. Fall back to the generic adapter when a domain is unknown.

Adapters are Stage 3 no-fetch:
- SubstackAdapter: parse provided title/snippet/meta only.
- TwitterAdapter: pass-through `tweet_text`/caption.
- GenericAdapter: excerpt-only fallback.
Registry determines the adapter by domain; gated by `FEATURE_DOMAIN_ADAPTERS=true`.

---
- Notes
- In-cluster only: `RAG_URL=http://rag:8088`. Host port is not published by default.
- API and Worker disable proxy inheritance for in-cluster RAG HTTP clients; timeouts configurable.
- Shared chunker lives in `services/shared/rag_chunks.py` and is used by both API reindex and Worker; step floor set to `max(1, size-overlap)`.
- Weekly pipeline groups assets under `weekly/<date>/...`, parses PDFs (pdfminer) and OCRs images in the worker (unified path), then upserts chart text.
- Bot-side OCR is disabled; the worker is the single source of truth for OCR text.

### OCR (worker-only, finalized)
- Input: original image (before normalization) for maximum glyph fidelity
- Preprocessing (tweet-tuned): grayscale → invert if dark → bilateral denoise → CLAHE → unsharp → upscale long-edge to 2200 → Otsu binarization
- Tesseract: tessdata_best via `TESSDATA_PREFIX=/usr/share/tesseract-ocr/5/tessdata`, `OCR_ENGINE_MODE=1` (LSTM), `OCR_PSM=6`, `preserve_interword_spaces=1`; optional second pass `PSM=4` if longer/cleaner
- Language: default `OCR_LANG=eng`; if `lang_hint` contains `zh`, use `chi_sim`
- Debug line (before OCR): `ocr: use_original=<bool> size=<WxH> lang=<lang> psm=<psm> oem=<oem> tess=<path>`

*Last updated: 2025-10-18*


