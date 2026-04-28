# PHASE-04-PR-IMPACT-ANALYSIS-VERIFICATION

## Required Checks
1. `I04-01` PR diff ingestion succeeds with changed-file list.
   Evidence: diff summary artifact.
2. `I04-02` Impact report includes routes, elements, scenarios, and risk scores.
   Evidence: `test_impact_report.json`.
3. `I04-03` Minimal regression subset is generated and executed.
   Evidence: selected test subset and run output.
4. `I04-04` Locator failures route into healing workflow automatically.
   Evidence: linked trace IDs and failure routing records.

## Pass Rule
All four checks must pass.

## Reviewer
CI quality owner.
