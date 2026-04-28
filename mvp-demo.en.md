# Python Self-Healing MVP Pseudocode (English Companion)

This file is the English companion of `mvp-demo.md`.

## MVP Goal
Implement the minimum healing loop:
1. Parse Java BDD failure
2. Collect runtime page evidence
3. Retrieve historical element profile
4. Generate locator candidates
5. Validate candidates in Playwright
6. Generate minimal Java patch
7. Rerun failed scenario
8. Persist healing report

## Core Model Set
- `FailedStep`
- `UiElementProfile`
- `PageSnapshot`
- `LocatorCandidate`
- `ValidationResult`
- `HealingPatch`

## Orchestration
The suggested orchestrator is LangGraph with explicit conditional routing:
- patch path if confidence and runtime checks pass
- triage path otherwise

## Safety Rules
- Never execute raw model output directly.
- Validate candidate uniqueness and actionability.
- Restrict patch scope to locator-safe areas.
- Require rerun evidence before marking success.
