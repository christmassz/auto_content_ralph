# Stage 4 — Website Search & Librarian Tools

## Goal
Enable the team to **find, tweak, and re-export** content quickly without heavy admin overhead.

## Scope
* `/content` search page with filters & pagination
* Detail view with lightweight edits (tags, timeframe, confidence, note)
* Merge-duplicates tooling
* Append-only audit trail

## Deliverables
1. Web UI pages:
   * List/search view (`/content`)
   * Detail/edit view
2. Merge duplicates workflow: mark canonical, archive dup, propagate to search/index.
3. Audit log table & display showing before→after, user, timestamp.

## Success Checks
* Common librarian edits can be completed in **<1 min**.
* Merge operation updates RAG index and selector candidates within 5 min.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Scope creep of librarian UI | Limit roles to admin + librarian; defer fancy workflows. |
| Performance issues on large datasets | Use server-side pagination & indexed queries. |

## Mapping to Project Status Board
* T9 – Inbox list page
* T10 – Detail & edit page
* T11 – Merge duplicates tooling
* T16 – Audit log & admin tools

### Headless browser fetcher (Playwright) – integration note

Add a small Playwright-based fetch service to handle JS-heavy pages when the basic link-fetcher fails. Jobs are dispatched on `link_fetch_browser` queue with per-domain rate limits and proxy support. Returned HTML/text is cleaned and stored like other sources. Use only for domains on the JS-heavy allowlist.

---
*Last updated: 2025-09-09*

## Telegram /s Search (Stage 4 QoL)

Goal: Let the team search naturally in Telegram with a cheap LLM normalizer and safe fallback.

### Flow
- User sends `/s <query>` in DM.
- Bot calls OpenAI-compatible endpoint (OPENAI_API_BASE/KEY) with strict JSON-only prompt (timeout ~1.5s).
- If normalizer returns a plan → validate/clip (Pydantic), else fallback to regex parser.
- Bot calls `GET /search?expand=true` with `top_k≈50–100` then client-filters by `ingest_ts` window and dedupes per `id`.
- Results are paginated 5 per page via inline buttons.

### Env
- `OPENAI_API_BASE`, `OPENAI_API_KEY` (required to enable normalizer)
- `NORMALIZER_MODEL` (default `gpt-4o-mini`)
- `NORMALIZER_TIMEOUT_MS` (default `1500`)

### Guardrails
- Pydantic Plan: `{query, date_gte, date_lte, source, top_k<=50, expand, mode}`
- Fix inverted dates; default `source=weekly`, `mode=auto`.
- Memo cache (20 min) of normalized plans; circuit breaker skips LLM after 3 timeouts for 2 min.
- Telemetry: one log line per `/s` with `norm_ms`, `fallback_used`, plan fields, `hit_count`.

### Optional API nicety
- `GET /search?mode=db` to bypass RAG when the plan asks for DB-first; default remains `auto`.

### Test Cases
- `/s mentions of gold vs bitcoin past 6 months` → weekly + 6‑month window; deduped.
- `/s 过去3个月 黄金 比特币` → same, Chinese; fallback works if LLM times out.
- `/s "risk parity"` → quoted phrase; no dates.
- `/s funding rate spikes source:telegram last 14 days` → telegram + 14‑day window.

