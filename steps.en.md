# AI-Assisted Self-Healing UI Test Execution Phases (English Companion)

This file is the English companion of `steps.md`.

## 0. 2026-04-28 Baseline Override (Highest Priority)

Execution baseline has been updated to a decoupled control-plane architecture:

1. Platform runs independently from serviced automation repositories.
2. Jenkins webhook is the primary trigger; platform can also trigger Jenkins jobs.
3. GitLab automation scope is branch + Draft MR only (no auto-merge to main).
4. Rollback policy uses `revert commit`; revert conflict is marked and stopped.
5. V1 handles locator failures only (no assertion semantic healing).
6. Timezone baseline is `Asia/Shanghai`.
7. One project may bind multiple Jenkins jobs; parallel execution must be traceable.

Execution source of truth:

- `V1_API_CONTRACT.zh-CN.md`
- `V1_DATABASE_DDL.zh-CN.md`
- `V1_LANGGRAPH_STATE_MACHINE.zh-CN.md`

When historical sections conflict with this baseline, use this baseline.

## Scope
The project targets a closed loop for legacy Java + BDD + Playwright automation:
- Frontend change impact analysis
- Runtime failure healing
- Java patch generation
- Scenario rerun
- Knowledge base feedback loop
- New-story test draft generation

## Phase Summary
1. Phase 00: Baseline binding between frontend and automation assets.
2. Phase 01: Locator healing MVP loop.
3. Phase 02: Runtime element inventory and knowledge ingestion.
4. Phase 03: Frontend AST + repo graph mapping.
5. Phase 04: PR-level proactive impact analysis.
6. Phase 05: Java patch governance and auditable approvals.
7. Phase 06: Story card to BDD/Java draft generation.
8. Phase 07: Enterprise controls, audit, masking, metrics.

## Key Principles
- LLM proposes candidates, Playwright runtime validates them.
- Keep patch scope minimal and auditable.
- Separate locator failures from business/data/environment failures.
- Persist both success and failure evidence for learning.
