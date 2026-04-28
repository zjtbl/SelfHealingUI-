# AGENTS.md

## Mission
Build an AI-assisted, auditable UI self-healing testing platform for:
- Java + BDD + Playwright automation assets
- React 18 + Ant Design 5 frontend changes

## Active Baseline (2026-04-28)
Use the following as current source of truth:
1. `V1_API_CONTRACT.zh-CN.md`
2. `V1_DATABASE_DDL.zh-CN.md`
3. `V1_LANGGRAPH_STATE_MACHINE.zh-CN.md`

Execution profile:
1. Platform is a decoupled control plane.
2. Jenkins webhook is the primary trigger.
3. V1 default mode is failure analysis only (no patch write).
4. V1.5 enables branch patch flow by feature flag.
5. Revert conflict must be marked and stopped.
6. Timezone baseline is `Asia/Shanghai`.

This repository is currently a design-and-planning workspace. Changes in this repo should prioritize architecture, planning, validation rules, and governance artifacts.

## Operating Model
Use a phase-gated workflow:
1. Explore
2. Plan
3. Implement in isolated scope
4. Verify with executable evidence
5. Review and decide L1/L2/L3 route
6. Persist artifacts to knowledge assets

## Prompt Structure (Required)
Every substantial task should include:
- Goal
- Context (files, routes, failing scenarios, logs)
- Constraints (safety, architecture, coding boundaries)
- Done-when criteria (tests/evidence that must pass)

## Hard Guardrails
- LLM proposes candidates; runtime validation decides.
- Do not treat business-flow changes as locator-only fixes.
- Do not weaken assertions to force green runs.
- Do not patch Cucumber business steps for L1 locator repair.
- Prefer Page Object / locator registry changes with minimal diff.
- Keep human approval for L2/L3 risk levels.

## Verification Requirements
A task is not done without evidence. Minimum evidence types:
- `failure_diagnosis.json`
- `locator_candidates.json`
- `validation_results.json`
- `java_patch.diff` (if patch is produced)
- `rerun_result.json`
- `healing_report.md`

## Skill Usage
Use local project skill when relevant:
- `.codex/skills/self-healing-ui/SKILL.md`

When adding new skills:
- Keep one skill focused on one job.
- Define clear trigger phrases.
- Define strict input/output contracts.
- Add scripts/assets only when they improve reliability.

## Project Map
Refer to:
- `PLAN.md` for phase roadmap.
- `PROJECT_STRUCTURE.md` for directory architecture.
- `TEST_STRATEGY.md` for validation strategy.
- `phases/` for phase-level execution specs.
- `verification/` for phase-level acceptance checks.

## Done Definition
A change is complete only when:
- It satisfies phase scope boundaries.
- Verification checks for that phase pass.
- Evidence artifacts are generated and reviewable.
- Risks are documented and routed to proper approval level.
