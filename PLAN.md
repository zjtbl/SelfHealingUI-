# PLAN.md

## Objective
Define the execution roadmap for a production-grade UI self-healing testing platform, based on existing project documents and agent best practices.

## Phase Roadmap
1. Phase 00: Baseline Binding
2. Phase 01: Locator Healing MVP
3. Phase 02: Runtime Element Inventory
4. Phase 03: Frontend AST and Repo Graph
5. Phase 04: PR Impact Analysis
6. Phase 05: Java Patch Governance
7. Phase 06: Story-to-Test Generation
8. Phase 07: Enterprise Governance and Metrics

## Delivery Rules
- Each phase must have:
  - A phase spec in `phases/`
  - A verification spec in `verification/`
  - Explicit entry and exit criteria
  - Required evidence artifacts

## Traceability Rules
- Keep every run tied to:
  - `trace_id`
  - `commit_sha`
  - `source_repo`
  - `tool_version`
  - `created_at`

## Risk Routing
- L1: locator/test contract/wait strategy changes
- L2: business assertion or flow semantics
- L3: environment, data, backend, permissions

## Immediate Next Milestones
1. Complete Phase 00 artifacts and verification evidence.
2. Implement Phase 01 workflow in an isolated pilot scope (1 page, 10-50 scenarios).
3. Produce first auditable healing report set.
