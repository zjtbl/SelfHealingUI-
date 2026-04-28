# EXECUTION_WORKFLOWS.md

## Workflow A: Baseline Binding
1. Bind frontend repo and automation repo.
2. Build route-page-scenario map.
3. Export initial locator inventory.
4. Validate one end-to-end baseline run.

## Workflow B: Failure Healing Loop
1. Parse failure log and classify failure type.
2. Collect runtime page evidence.
3. Retrieve historical element profile.
4. Generate locator candidates.
5. Validate candidates in browser runtime.
6. Generate minimal Java patch.
7. Rerun impacted scenario.
8. Persist artifacts and report.

## Workflow C: PR Impact Analysis
1. Parse frontend diff.
2. Resolve affected components/routes/elements.
3. Resolve impacted scenarios from graph bindings.
4. Compute risk score and select minimal regression set.
5. Execute subset and route failures to healing loop.

## Workflow D: Story-to-Test Draft
1. Parse story and acceptance criteria.
2. Link new/changed routes and runtime page evidence.
3. Recall similar historical scenarios.
4. Generate BDD draft first.
5. Reuse existing step definitions where possible.
6. Generate Java draft only for missing parts.
7. Require human review before merge.
