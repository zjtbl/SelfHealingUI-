# PHASE-01-LOCATOR-HEALING-MVP

## Objective
Deliver the minimal healing loop:
failure -> candidate locator -> runtime validation -> patch -> rerun -> report.

## Inputs
- Failure logs and stack traces
- Page evidence (DOM/a11y/screenshot)
- Baseline route and locator mappings

## Outputs
- Structured candidate set
- Runtime validation results
- Minimal Java locator patch diff
- Rerun outcome and healing report

## Exit Criteria
- At least one locator-level failure is healed with rerun pass.
- Artifacts are persisted and reviewable.

## Out of Scope
- Fully automatic merge to main branch.
