# UI Self-Healing Knowledge Base and RAG Design (English Companion)

This file is the English companion of `konwledge-rag.md`.

## Goal
Build a testing intelligence hub that links:
- business intent
- UI elements
- frontend implementation
- automation steps
- execution evidence
- repair history

## Data Layers
1. Structured database for facts and versioned records.
2. Graph relationships for impact traversal.
3. Vector collections for semantic recall.
4. Object storage for large artifacts (trace, screenshot, DOM).

## Self-Healing Retrieval Strategy
Use multi-channel recall and rerank:
- exact structured recall
- graph impact recall
- similar failure recall
- semantic element recall
- visual evidence recall

## Acceptance Focus
- Traceability from locator to scenario and frontend component.
- PR diff to impacted scenario mapping.
- Clear separation of successful healing, rejected healing, and environment failures.
