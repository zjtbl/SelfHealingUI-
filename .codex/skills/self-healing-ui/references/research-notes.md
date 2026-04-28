# Research Notes

Use these notes when making recommendations for AI-assisted self-healing UI testing.

## Playwright

- Prefer built-in locators: role, text, label, placeholder, alt text, title, test id.
- Role locators align with accessibility semantics and should usually include accessible name.
- Test ids are resilient when text or roles change, but they are not user-facing; use them as explicit test contracts for business-critical or i18n-sensitive controls.
- Strictness errors are useful signals: a locator that matches multiple elements needs better scoping, not `first()` by default.
- Playwright MCP accessibility snapshots are useful for LLM page understanding because they provide structured accessible elements and refs, instead of relying only on screenshots.

## Healenium

- Healenium is the closest open-source reference for Selenium-style locator healing.
- Reuse its concepts: historical selectors, proxy/wrapper interception, healing reports, persisted selector history, scoring.
- Do not copy the architecture directly for Java Playwright. Use Playwright runtime validation and Java code patching instead of Selenium driver proxying.

## Repo Map / AST

- Repo maps use AST or Tree-sitter to extract symbols and relationships from source code.
- For this domain, adapt the idea to frontend and automation assets:
  - React route/component map.
  - Ant Design component graph.
  - Java BDD feature/step/Page Object/locator map.
  - commit impact graph.

## Recommended Design Bias

- Rules and Playwright decide whether a candidate is valid.
- LLM proposes candidates, explains intent, and drafts patches.
- The knowledge base stores facts, versions, and evidence.
- The vector database retrieves similar elements, steps, failures, and repair cases.
- Human review remains mandatory for business semantics, assertion changes, and high-risk flows.

## Useful Public References

- Playwright locators: https://playwright.dev/docs/locators
- Playwright Java locators: https://playwright.dev/java/docs/locators
- Playwright MCP snapshots: https://playwright.dev/mcp/snapshots
- Healenium: https://github.com/healenium/healenium
- Healenium docs: https://healenium.io/docs/overview
- Aider repository map: https://aider.chat/docs/repomap.html
