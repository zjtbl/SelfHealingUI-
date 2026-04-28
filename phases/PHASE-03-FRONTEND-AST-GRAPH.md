# PHASE-03-FRONTEND-AST-GRAPH

## Objective
Build static understanding of frontend components and map them to testing assets.

## Inputs
- React/AntD source tree
- Routing configuration
- Existing route-element-scenario mapping

## Outputs
- Component graph snapshot
- Route-to-component-to-element bindings
- Early locator fragility flags

## Exit Criteria
- Changed files can be mapped to impacted routes and elements.
- Missing test contract signals (`data-testid`, role/label quality) are detectable.

## Out of Scope
- Full semantic correctness of business logic changes.
