# TEST_STRATEGY.md

## Strategy Objective
Guarantee that self-healing changes are trustworthy, reproducible, and reviewable.

## V1 Execution Policy
1. V1 default is analysis-only (no patch write).
2. V1.5 write-flow must be explicitly enabled per project.
3. Branch CI can be auto-triggered when write-flow is enabled.
4. Merge to main is always human-triggered.

## Validation Layers
1. Static checks
2. Runtime UI validation
3. Patch safety checks
4. Scenario rerun checks
5. Governance checks

## Static Checks
- Locator inventory extraction completeness
- Frontend diff parsing completeness
- Phase config schema validity

## Runtime UI Validation
- Candidate locator uniqueness
- Visibility and actionability
- Action replay pass (click/fill/assert)
- Portal/scope correctness for AntD overlays

## Patch Safety
- Minimal diff policy
- Allowed edit surface policy
- No assertion weakening
- No business step semantic drift

## Rerun Validation
- Single-scenario rerun pass
- Optional impacted-set rerun pass
- Artifact bundle completeness

## Governance Validation
- L1/L2/L3 classification present
- L2/L3 must not enter automatic locator patch flow
- Audit fields present in each run record
- Revert conflict is marked and stopped

## Exit Rule
A phase is accepted only when its verification file in `verification/` is fully satisfied.
