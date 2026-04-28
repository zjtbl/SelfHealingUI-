# AI-Assisted Self-Healing UI Test Execution Phases (English Companion)

This file is the English companion of `steps.md`.

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
