# SYSTEM_ARCHITECTURE.md

## Architecture Style
Hybrid architecture:
- Rules engine for deterministic gates
- LLM agents for intent reasoning and candidate generation
- Playwright runtime for browser-grounded validation
- Knowledge assets for longitudinal memory and traceability

## Layer Model
1. Trigger Layer
2. Decision Layer (orchestrator/state graph)
3. Healing Layer (analyzer/healer/patcher)
4. Execution Layer (Java + BDD + Playwright runner)
5. Logging Layer (run state + diagnosis records)
6. Knowledge Layer (structured + vector + object artifacts)

## Core Flows
- Proactive flow: PR diff -> impact prediction -> minimal regression set
- Reactive flow: failure -> candidate generation -> runtime validation -> patch -> rerun

## Trust Boundaries
- LLM output is untrusted until runtime verification passes.
- Patch application is untrusted until rerun evidence passes.
- L2/L3 changes are untrusted until human approval.

## Data Domains
- Structured store: pages, elements, scenarios, runs, healing events
- Vector store: semantic recall for steps/elements/failures
- Artifact store: screenshot, trace, DOM, a11y snapshot, patch diff

## Reliability Principles
- One trace ID per run
- Idempotent evidence writes
- Explicit retry strategy
- Deterministic failure classification
