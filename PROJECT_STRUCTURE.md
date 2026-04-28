# PROJECT_STRUCTURE.md

## Top-Level Layout
```text
SelfHealingUI/
  AGENTS.md
  AGENT.md
  SKILLS.md
  PLAN.md
  PROJECT_STRUCTURE.md
  TEST_STRATEGY.md
  docs/
    architecture/
    workflows/
  phases/
  verification/
    templates/
  config/
  platform/
    orchestrator/
    agents/
    scanners/
    healing/
    runners/
    storage/
    models/
    prompts/
  tests/
    contract/
    integration/
    e2e/
  artifacts/
```

## Directory Responsibilities
- `docs/architecture/`: architecture decisions, data boundaries, trust boundaries.
- `docs/workflows/`: process flows from trigger to evidence.
- `phases/`: phase-specific execution specs.
- `verification/`: phase-specific acceptance checks.
- `config/`: machine-readable policies and project bindings.
- `platform/`: planned implementation modules.
- `tests/`: validation code layers (future implementation).
- `artifacts/`: runtime evidence bundles.

## Naming Convention
- English file: `NAME.md`
- Chinese counterpart: `NAME.zh-CN.md`

## Ownership Convention
- Strategy files: test architecture owner
- Config templates: platform owner
- Verification files: QA governance owner
