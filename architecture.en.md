# Java + BDD + Playwright Self-Healing Architecture (English Companion)

This file is the English companion of `架构.md`.

## Recommended Architecture
Use a hybrid model:
- deterministic rules engine
- LLM agents for reasoning and candidate drafting
- Playwright runtime validation as final gate
- knowledge base for persistent memory and traceability

## Main Agent Responsibilities
- Change Analyzer
- Impact Analyzer
- Failure Collector
- Locator Healer
- Runtime Validator
- Java Patch Agent
- Review Agent

## CI Integration
- PR stage: impact-driven subset test and healing loop on locator failures.
- Mainline stage: broader regression and feedback ingestion.
- Nightly stage: drift checks and fragile locator reduction actions.

## Governance Boundaries
- L1: locator/test contract/wait strategy
- L2: assertion or business semantics
- L3: environment/data/backend failures
