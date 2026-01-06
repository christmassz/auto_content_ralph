## User Manual (Simple)

### Launch

```bash
./build.sh
```

This starts API, Worker, RAG, Redis, Postgres, MinIO, Gateway, and the RSS poller. It also issues an admin token into `bot.jwt` and wires defaults.

---

### Add content

#### 1) Telegram (fastest)
- DM your bot: send content (text or image caption) → tap a timeframe → add a subject if prompted → Confirm → Saved.
- Or in a group with the librarian: reply to the wizard prompts (timeframe → subject → content → Confirm).

Prereqs: set `TG_BOT_TOKEN` in `.env` before `./build.sh`.

#### 2) RSS (hands‑free)
- Edit `.env`:
  - `RSS_FEEDS_FILE=/app/rss_feeds.txt` (mounted from `ops/rss_feeds.txt`)
  - or fallback: `RSS_FEEDS=<comma-separated feed URLs>`
  - `RSS_INTERVAL_SEC=86400` (once per day)
- Example:
  - Add `https://www.jl.capital/insights.xml` to `ops/rss_feeds.txt` ([feed](https://www.jl.capital/insights.xml))
- Mount in compose: `./ops/rss_feeds.txt:/app/rss_feeds.txt:ro`
- Restart RSS service (or re-run `./build.sh`). New items post to `/ingest/web` automatically.

#### 3) Website (direct API)
Minimal example (replace TOKEN with `/auth/token` value or use `bot.jwt` for automations):

```bash
TOKEN='<access_token>'
curl -s -X POST "http://localhost:8000/ingest/web" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
        "subject":"Sample subject",
        "timeframe":"daily",
        "text":"Body text or leave empty if you only have links",
        "links":["https://example.com/page"],
        "media":[],
        "provenance":{"user_id":"admin"}
      }'
```

#### 4) Twitter/X
- Use Telegram (paste the tweet link) or call `/ingest/web` with the tweet URL in `links`. The Twitter adapter passes through `tweet_text`/caption when present.

---

### Search

```bash
curl -s "http://localhost:8000/search?q=bitcoin&top_k=5&expand=true" | jq .
```

Returns ranked items; when RAG is healthy and `FEATURE_RAG=true`, `rag_used` is `true`.

---

### Reindex (admin)

```bash
TOKEN='<access_token>'
curl -s -X POST "http://localhost:8000/admin/reindex?recent=200" -H "Authorization: Bearer $TOKEN"
```

---

### Notes
- In-cluster RAG URL is `http://rag:8088` (no host port exposed). API/Worker already use it.
- RSS poller de-duplicates via a stable key per entry; it ingests what the feed exposes.
- Worker performs OCR, summarization, scoring, and RAG indexing automatically after ingest.
- See also: Stage 3.5 — Content Ingestion QoL (`Docs/Stage3_5_Content_Ingestion.md`).

### Backfill later (no re-upload)
- Originals are preserved in MinIO. When you add PDF/charts extractors, run a one-time backfill over existing items to enrich text, then reindex RAG.
- Prefer bumping `RAG_INDEX_VERSION` before reindex, then delete old version and compact.


