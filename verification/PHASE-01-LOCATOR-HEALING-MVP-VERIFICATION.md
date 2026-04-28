# PHASE-01-LOCATOR-HEALING-MVP-VERIFICATION

## Required Checks
1. `H01-01` Failure parser extracts scenario, step, locator, and error type.
   Evidence: `artifacts/<trace_id>/failure_diagnosis.json`.
2. `H01-02` At least 3 locator candidates are generated in structured format.
   Evidence: `artifacts/<trace_id>/locator_candidates.json`.
3. `H01-03` Candidate validation includes uniqueness and actionability checks.
   Evidence: `artifacts/<trace_id>/validation_results.json`.
4. `H01-04` Patch is minimal and limited to allowed surfaces.
   Evidence: `artifacts/<trace_id>/java_patch.diff`.
5. `H01-05` Rerun passes for target failed scenario.
   Evidence: `artifacts/<trace_id>/rerun_result.json`.
6. `H01-06` Healing report is generated and reviewable.
   Evidence: `artifacts/<trace_id>/healing_report.md`.

## Pass Rule
All six checks must pass.

## Reviewer
Automation lead + QA lead.
