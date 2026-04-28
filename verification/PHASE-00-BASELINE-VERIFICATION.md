# PHASE-00-BASELINE-VERIFICATION

## Required Checks
1. `B00-01` Project binding exists and is complete.
   Evidence: `config/project_binding.example.yaml` or project-specific binding file.
2. `B00-02` Route-page-scenario mapping exists for pilot scope.
   Evidence: route mapping artifact under `artifacts/<trace_id>/`.
3. `B00-03` Locator inventory is generated with file and line references.
   Evidence: `artifacts/<trace_id>/locator_inventory.json`.
4. `B00-04` One smoke scenario runs successfully with evidence bundle.
   Evidence: `rerun_result.json`, screenshot/trace links.

## Pass Rule
All four checks must pass.

## Reviewer
QA architecture owner.
