## Stage 3.5 — Content Ingestion (Quality-of-Life)

Goal: Make adding content effortless and reliable via auto-inputs and simple controls, building on Stage 3.

### Scope
- File-based RSS feeds list (`ops/rss_feeds.txt`) and daily polling
- Simple website inputs (manual via `/ingest/web`; optional websites.txt future hook)
- Telegram librarian flows (DM and group wizard)
- Smooth operator experience: minimal steps to add/verify sources and health

### What’s included (now)
- RSS poller reads `RSS_FEEDS_FILE=/app/rss_feeds.txt`; mount `./ops/rss_feeds.txt:ro`
- `/ingest/web` accepts direct posts (with links and media)
- Telegram acceptor and group librarian flows
- Worker enriches, OCRs, summarizes, scores, and indexes to RAG automatically

### Operator checklist
- [ ] Add feeds in `ops/rss_feeds.txt` (one per line, `#` comments allowed)
- [ ] Set in `.env`: `RSS_FEEDS_FILE=/app/rss_feeds.txt`, `RSS_INTERVAL_SEC=86400`
- [ ] `./build.sh` or `docker compose up -d rss`
- [ ] (Optional) Set `FEATURE_LINK_FETCHER=true` to enrich link text
- [ ] Issue admin token (`/auth/token`) and bot token (`scripts/issue_token.py`)
- [ ] Verify ingestion end-to-end: RSS → /ingest/web → worker logs → RAG → /search

### Backfill later without re-upload
- Originals (images/docs) are preserved in MinIO. When new extractors (PDF text, chart digitizer) are added, run a one-time backfill over existing items to enrich `raw_text` (or a new field), then reindex RAG.
- Reindex options:
  - Reuse current version: `POST /admin/reindex?since=1970-01-01`
  - Prefer new version: bump `RAG_INDEX_VERSION`, reindex, then `/admin/rag/delete` old version → `/admin/rag/compact`.

### Health and troubleshooting
- Logs: `docker compose logs -f rss | worker | api | rag`
- RSS sanity: `curl -sSL <feed-url> | head -n 20`
- Search: `curl -s "http://localhost:8000/search?q=test&top_k=5&expand=true"`

### Nice-to-haves (optional)
- Websites file (`ops/websites.txt`) + daily web ingester (future)
- Per-domain throttles and robots.txt checks (web ingester)
- Metrics: new items/day, duplicates skipped, fetch errors


