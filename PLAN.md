# PLAN.md

## Objective
Define the execution roadmap for a production-grade UI self-healing testing platform, based on existing project documents and agent best practices.

## Execution Baseline
1. V1 scope: standalone control plane + Jenkins webhook ingestion + failure analysis + decision recording.
2. V1.5 scope: locator branch-healing flow enabled by feature flag.
3. GitLab merge to main is always manual.

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
1. Complete webhook ingestion, payload enrichment, and artifact fetching for one project.
2. Complete locator-failure classification and decision recording in analysis-only mode.
3. Enable optional V1.5 branch flow in a controlled pilot and validate revert behavior.
