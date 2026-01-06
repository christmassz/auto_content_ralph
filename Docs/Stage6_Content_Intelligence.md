# Stage 6 — Content Intelligence

## Goal
Evolve “quality” from a heuristic proxy to a **data-driven score** informed by real feedback.

## Scope
* CM feedback capture (👍 / meh / skip)
* Labelled dataset creation
* Heuristic/ML scoring v2 with A/B toggle in selector
* “Aged-well” bias for past calls with proven outcomes

## Deliverables
1. Feedback capture via TG FYI message inline buttons; stored per content item.
2. Training pipeline that produces updated weights or small model artifact.
3. Selector mod that can **toggle between v1 (rules) and v2 (learned) scores**.
4. Optional novelty boost for items with strong “aged-well” signals.

## Success Checks
* Selector A/B test shows statistically significant improvement in CM publish rate **and** live engagement.

## Risks & Mitigations
| Risk | Mitigation |
| --- | --- |
| Model drift degrading quality | Retrain on rolling window; monitor score distributions. |
| Sparse feedback data | Keep hybrid (rules + learned) scoring until coverage adequate. |

## Mapping to Project Status Board
* Extends T8 scoring tasks
* Adds small analytics/ML job under processing pipeline

---
*Last updated: 2025-09-09*
