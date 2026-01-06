# Stage 3 — Search & RAG Plug-in

## Goal
Introduce smart retrieval & novelty scoring **without breaking** Stage 1/2 flows.

## Scope
* Stand-alone RAG-Anything service (FAISS index)
* Worker that indexes on `content.updated` events
* `/search` API proxy that toggles via `FEATURE_RAG`
* Selector novelty boost via `IRetrieval.rank`
* Worker-only OCR (unified across sources)

## Deliverables
1. **RAG service** with `/upsert`, `/query`, cosine-normalized search, and richer `/healthz` (version, model, uptime); index persisted under `rag_index/`.
2. Admin backfill `POST /admin/reindex` that normalizes text, chunks (≈900/150), enriches metadata (`chunk_id`, `rag_index_version`, `content_id`), batches with retries, and upserts to RAG.
3. API endpoint `GET /search` proxies to RAG when `FEATURE_RAG=true`; `expand=true` enriches results from DB.
4. Worker-only OCR finalized; adapters (Substack/Twitter/Generic) integrated pre-summarization.
5. Worker-triggered RAG indexing on content updates (chunking + batched `/upsert`).
6. Fallback to DB/rule-based when RAG errors; `rag_used=false` returned.
7. Idempotent upsert via FAISS `IndexIDMap2` with stable 63-bit IDs from `chunk_id`.
8. Maintenance endpoints: `/delete`, `/compact`, `/rebuild`; active index version persisted and exposed.
9. Bridged-only networking; in-cluster `RAG_URL=http://rag:8088`. API disables proxy inheritance for in-cluster RAG calls.

## Success Checks
* `/search` returns relevant results and `rag_used=true` when RAG answers; returns `rag_used=false` on fallback with non-error 200.
* Worker logs: `rag_upsert_ms=<n> chunks=<m>` per processed content.
* RAG `/healthz` shows increasing `items` and correct `rag_index_version`.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Re-index storms during model/param change | Use `rag_index_version`; build side-by-side then flip pointer. |
| Increased infra complexity | Keep RAG as isolated service with clear health checks & backups. |
| RAG downtime causing search 500s | Automatic fallback to Postgres search path; alert triggered. |

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

### OCR (worker-only, finalized)
- Input: original image (before normalization) for maximum glyph fidelity
- Preprocessing (tweet-tuned): grayscale → invert if dark → bilateral denoise → CLAHE → unsharp → upscale long-edge to 2200 → Otsu binarization
- Tesseract: tessdata_best via `TESSDATA_PREFIX=/usr/share/tesseract-ocr/5/tessdata`, `OCR_ENGINE_MODE=1` (LSTM), `OCR_PSM=6`, `preserve_interword_spaces=1`; optional second pass `PSM=4` if longer/cleaner
- Language: default `OCR_LANG=eng`; if `lang_hint` contains `zh`, use `chi_sim`
- Debug line (before OCR): `ocr: use_original=<bool> size=<WxH> lang=<lang> psm=<psm> oem=<oem> tess=<path>`

---
*Last updated: 2025-09-09*


