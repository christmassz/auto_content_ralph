# Stage 2 — Quality & Safety Nets

## Goal
Reduce low-quality or duplicate picks **without** introducing RAG or heavy UI.

## Scope
* Near-duplicate detection (text SimHash, image pHash)
* Lightweight scoring (freshness & quality proxies)
* “Unverified” handling & emergency override switches

## Deliverables
1. **Dedup++**:
   * SimHash for text; pHash for images; store `similarity_to_recent`.
2. **Scoring** formulas:
   * `freshness_score = exp(-age_days / half_life_by_timeframe)`
   * `quality_proxy` = length norm + banned-term check + source weight
   * `selector_score = 0.6*freshness + 0.4*quality_proxy`
3. Allow ≤1 `unverified` item/day with hedged caption.
4. Simple overrides:
   * `.skip_today` flag
   * `exclude_ids` list in config
5. **Per-curator throttle** enforced by Redis token bucket (e.g., 1 msg per 10 s burst, 30 per day).

## Success Checks
* Duplicate rate drops measurably (baseline vs Stage 1).
* CM thumbs-down rate declines week-over-week.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| False positives on near-dup detection | Set conservative thresholds; log candidate pairs for manual review. |
| Scoring weights mis-tuned | Keep coefficients in config; iterate with metrics. |

## Mapping to Project Status Board
* T8.x – Processing pipeline jobs
* T12.1–12.3 – Selector rule refinements
* T19.2 – Validation caps & overrides

### RSS Integration (Docker poller)

We support ingesting website RSS feeds during Stage 2 without API/schema changes by running a lightweight poller container that posts normalized entries to the existing `/ingest/web` endpoint.

Setup steps:
1. Fill `.env` with:
   - `RSS_FEEDS=comma,separated,feed,urls`
   - `BOT_JWT=<token>` (issue once and paste)
   - Optional: `RSS_DEFAULT_TIMEFRAME`, `RSS_INTERVAL_SEC`, `RSS_MAX_TEXT`, `API_BASE`
2. Build and start the service:
```
docker compose up -d --build rss
```
3. Verify logs:
```
docker compose logs -f rss
```

Notes:
- The poller uses an idempotency key derived from feed URL + GUID/link + published date to avoid duplicates.
- Items are tagged under `provenance.user_id = "rss:<feed_shortname>"` and flow through dedup/scoring like other sources.
- Per-user/day limits apply; split busy feeds across identities if needed.

### Link fetcher (Stage 2 scope)

Add a lightweight worker step to enrich RSS items that include `links[]` by fetching the canonical article and extracting `title`, `main_text`, `lead_image`, and OpenGraph metadata using readability/trafilatura/newspaper3k. Persist extracted text into `content_items.raw_text` and attach images to `media_assets`. This runs after initial ingest and before similarity/scoring.

### OCR extraction (Stage 2 optional)

- Feature flags: `FEATURE_OCR` and `FEATURE_OCR_SUMMARIZE` (default off).
- Worker runs OCR on normalized images (Tesseract via pytesseract) and appends text into `content_items.raw_text` with an `[OCR]` delimiter when non-empty.
- Optional summarization: when `FEATURE_OCR_SUMMARIZE=true`, generate a one-line caption from OCR text (reusing the existing summarizer) and set it as the item caption.
- Env knobs: `OCR_LANG` (e.g., `eng+chi_sim`), `OCR_MAX_CHARS`, `OCR_SUMMARY_LANG`, `OCR_SUMMARY_MAX_CHARS`.
- No schema change required; media normalization and scoring proceed as before.

---
*Last updated: 2025-09-09*
