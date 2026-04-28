# PHASE-04-PR-IMPACT-ANALYSIS

## Objective
Predict impact from frontend diffs and execute only the minimal relevant regression subset.

## Inputs
- PR/commit diff
- AST graph and route bindings
- Scenario mapping and historical risk signals

## Outputs
- Impact report (routes, elements, scenarios)
- Risk score per impacted area
- Selected regression test subset

## Exit Criteria
- At least one PR run uses impact-driven subset selection.
- Report quality is sufficient for reviewer decision-making.

## Out of Scope
- Replacing full regression policy for all release gates.
