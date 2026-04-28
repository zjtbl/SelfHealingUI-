# PHASE-03-FRONTEND-AST-GRAPH-VERIFICATION

## Required Checks
1. `A03-01` AST scanner resolves changed files to components.
   Evidence: component graph snapshot artifact.
2. `A03-02` Route bindings for changed components are available.
   Evidence: route-component mapping artifact.
3. `A03-03` Locator fragility risk signals are produced.
   Evidence: risk fields in output report.
4. `A03-04` Missing contract signals (`data-testid`/role/label) are flagged.
   Evidence: static analysis warning list.

## Pass Rule
All four checks must pass.

## Reviewer
Frontend testability owner.
