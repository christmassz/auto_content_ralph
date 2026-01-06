# Acceptance Criteria & Testing

This file defines the acceptance criteria and verification scripts for the Ralpha project.

## Stage 0 - Preflight & Contracts

**Acceptance Script:** `./scripts/check_stage0.sh`

Requirements:
- [ ] Health endpoints responding
- [ ] Database schema initialized
- [ ] Feature flags baseline established
- [ ] CI pipeline functional

## Stage 1 - Walking Skeleton MVP

**Acceptance Script:** `./scripts/check_stage1.sh`

Requirements:
- [ ] End-to-end ingest → process → export flow working
- [ ] Telegram bot integration functional
- [ ] Basic content processing pipeline operational
- [ ] Export to content bank successful

## Manual Verification Gates

### Stage 1 Soak Run
**Manual bead:** "Stage 1 Soak Run — 5-day daily export + publishable rate tracking"

This bead's DoD is operational, not code:
- Run the system daily for 5 days
- Collect results
- Record publishable rate
- Decide what must change

*This bead requires human verification and cannot be completed by automated processes.*