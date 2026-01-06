# Stage 7 — Autopost & Multichannel

## Goal
Reduce human friction once quality is trustworthy by **safe autoposting** and multi-channel exports.

## Scope
* WeChat autopost toolchain (or one-click clipboard helper if API unavailable)
* Optional Telegram Channel / X (Twitter) cross-posts
* Per-channel templates (caps, formats)

## Deliverables
1. **Autopost module** with retry & failure logs; toggle via `EXPORT_AUTOPOST` flag.
2. Channel-specific template renderer (WeChat, TG, X) with platform caps.
3. Rollback path: immediate unpublish or delete option.
4. CM “pause” switch that stops autopost and reverts to manual copy-paste.

## Success Checks
* Zero missed posts during test window.
* CM can pause / resume with a single flag change.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Platform API changes breaking autopost | Isolate each platform in adapter; graceful degradation to manual export. |
| Accidental spam / duplicate posting | Dry-run mode + explicit confirmations during rollout. |

## Mapping to Project Status Board
* Extends T14 and T13 tasks
* Adds channel-specific exporter subtasks

---
*Last updated: 2025-09-09*
