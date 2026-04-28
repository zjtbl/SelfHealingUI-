---
name: self-healing-ui
description: Design and implement AI-assisted self-healing UI automation for Java BDD Playwright projects. Use when analyzing legacy Java/Cucumber/BDD UI test code, scanning Playwright locators and page elements, designing locator healing, building UI element knowledge bases or vector search, generating Java locator patches, planning React/Ant Design impact analysis, or writing self-healing test rollout steps.
---

# Self-Healing UI

## Operating Model

Treat self-healing as a verified engineering harness, not as direct LLM code mutation.

Use this loop:

```text
orient to repo state
  -> choose workflow mode
  -> scan assets
  -> classify risk or failure
  -> collect runtime evidence
  -> retrieve historical context
  -> generate locator or patch candidates
  -> validate with Playwright
  -> patch only safe code regions
  -> rerun impacted tests
  -> persist evidence and decision
```

## Current Delivery Profile (2026-04-28)

Use this profile as the active execution baseline:

1. The platform is a standalone control plane and must stay decoupled from serviced automation repos.
2. Primary trigger is Jenkins webhook callback to platform API.
3. Platform also actively triggers Jenkins jobs when required.
4. GitLab automation scope: create branch, commit patch, create Draft MR, query MR changes, revert commit.
5. No automatic merge to main branch.
6. V1 healing scope is locator-only failures; assertion semantic changes are out of scope.
7. Revert conflict handling: mark failure and stop automation.
8. Timezone baseline: `Asia/Shanghai`.
9. One project may configure multiple Jenkins jobs; parallel runs must be traceable by `event_id` and `run_id`.

Always separate:

- **Intent**: the business action or assertion.
- **Element**: stable `element_uid` and semantic profile.
- **Locator**: versioned implementation detail.
- **Evidence**: DOM, accessibility snapshot, screenshot, trace, logs.
- **Patch**: minimal Java or frontend diff.

## Session Start Protocol

At the start of every substantial task:

1. Confirm `pwd` and stay inside the project.
2. Read `steps.md`, `skill-self-think.md` if present, and the relevant project config.
3. Check recent progress artifacts if present: `docs/exec-plans/active/`, `docs/exec-plans/completed/`, `self-healing-progress.md`, `artifacts/`.
4. Select one workflow mode and one primary output artifact.
5. State the validation command or evidence required before calling the task done.

Do not mark work complete only because a document or patch was generated. Completion requires evidence.

## Workflow Modes

Route work by intent before acting:

- **Product challenge**: refine scope, success metrics, and user impact for the self-healing platform.
- **Engineering plan review**: inspect architecture, data flow, testability, failure modes, and rollout risk.
- **Baseline import**: bind frontend repo, Java automation repo, routes, features, CI commands, and environments.
- **Locator scan**: inventory Java Playwright locators and classify fragility.
- **Runtime scan**: use Playwright to collect DOM, accessibility snapshots, screenshots, traces, and element profiles.
- **Healing loop**: diagnose locator-level failures, generate candidates, validate with Playwright, patch Java, rerun.
- **Impact analysis**: map frontend diffs to route/component/element/scenario risk.
- **Review**: find production risks, false healing, weak gates, security/data exposure, and missing tests.
- **QA**: exercise the app as a user with browser automation and attach evidence.
- **Ship**: verify gates, summarize artifacts, and prepare PR or release notes.
- **Retro**: convert failures, manual fixes, and review feedback into durable harness rules.

## Harness Rules

Use these engineering rules:

- Keep the project knowledge base repository-local and versioned. External memory is secondary.
- Treat top-level docs as a map, not an encyclopedia. Put details behind linked files.
- Prefer structured state files over free-form prose for pass/fail tracking.
- Work one verifiable slice at a time.
- Make browser validation a first-class gate for UI behavior.
- Encode repeated review feedback into policy files, scripts, linters, or skill instructions.
- Keep all generated artifacts tied to `trace_id`, `commit_sha`, `source_repo`, and `tool_version`.
- Use completion statuses: `DONE`, `DONE_WITH_CONCERNS`, `BLOCKED`, `NEEDS_CONTEXT`.

## First Pass On A Repo

1. Locate Java test project metadata: `pom.xml`, `build.gradle`, Cucumber features, step definitions, Page Objects, Playwright Java imports.
2. Inventory locators from Java code, including file, line, method, locator expression, action, and risk.
3. Locate frontend metadata if present: React routes, TSX/JSX pages, Ant Design imports, `data-testid`, `aria-label`, form labels, placeholders, button text.
4. Build or request a project binding when repos are separate:

```yaml
frontend_repo: ../frontend
automation_repo: ../e2e-java
base_url: https://test.example.com
test_command: mvn test -Dcucumber.filter.tags="@smoke"
routes:
  login:
    path: /login
    source_files:
      - src/pages/LoginPage.tsx
    feature_files:
      - src/test/resources/features/Login.feature
```

Recommended repository harness files:

```text
AGENTS.md
docs/harness/CORE_BELIEFS.md
docs/harness/WORKFLOWS.md
docs/harness/QUALITY_SCORE.md
docs/harness/RELIABILITY.md
docs/harness/SECURITY.md
docs/exec-plans/active/
docs/exec-plans/completed/
config/project_binding.yaml
config/locator_policy.yaml
config/repair_policy.yaml
config/env_policy.yaml
self_healing_features.json
self-healing-progress.md
scripts/init.sh
scripts/verify.sh
```

## Locator Policy

Prefer stable user-facing or explicit contract locators:

1. `getByRole` with accessible name.
2. `getByLabel` for form controls.
3. `getByPlaceholder` when placeholder is stable enough.
4. `getByTestId` for core business controls and i18n-sensitive text.
5. Scoped locators inside dialogs, list items, rows, cards, forms, or tables.
6. CSS/XPath only as fallback; mark as fragile.

Reject or require review for:

- Absolute XPath.
- Dynamic CSS classes.
- Random/generated IDs.
- `nth()` without a stable parent scope.
- Locator changes that weaken assertions or skip business validation.
- Unverified LLM-generated locator code.

## Runtime Validation

Do not accept candidate locators until a real browser validates them.

For each candidate verify:

- Match count is exactly 1 unless the step intentionally targets a collection.
- Element is visible.
- For click: enabled and trial-clickable.
- For fill: editable.
- For assertions: expected text/visibility can be observed.
- Candidate works in the same page state as the failed step.

Use Playwright trace, DOM, accessibility snapshot, screenshot, console logs, and network failures as diagnosis evidence.

## Failure Classification

Only L1 failures are eligible for automatic patch generation.

L1, locator-level:

- `LOCATOR_NOT_FOUND`
- `LOCATOR_NOT_UNIQUE`
- `ELEMENT_NOT_VISIBLE`
- `ELEMENT_NOT_ENABLED`
- `ELEMENT_OBSCURED`

L2, human review:

- assertion meaning changed
- business flow changed
- expected copy changed with product approval
- new mandatory field or permission flow appeared

L3, block and report:

- backend unavailable
- network/proxy failure
- test data missing
- login/session/permission failure
- environment health failure

## Java Patch Rules

Patch the smallest safe surface:

- Prefer Page Object method or locator registry changes.
- Avoid changing Cucumber step business flow.
- Preserve project style and imports.
- Prefer AST-aware patching with JavaParser or tree-sitter-java for production.
- MVP may do exact replacement only when old locator appears once in the target Page Object method.
- Never add unbounded retries, unconditional `force`, sleeps as a default fix, or weaker assertions.

Every patch must include:

- old locator
- new locator
- reason
- validation result
- rerun result
- review level
- rollback note

## Knowledge Base Shape

Maintain both structured and semantic storage.

Structured entities:

- `ui_page`
- `ui_component`
- `ui_element`
- `locator_version`
- `test_asset`
- `step_element_binding`
- `execution_run`
- `execution_observation`
- `healing_event`
- `review_decision`

Vector documents:

- element semantic profile
- BDD step and scenario summary
- failure diagnosis and repair case
- frontend component summary
- story or acceptance criteria summary

Object artifacts:

- screenshots
- element crops
- trace files
- DOM HTML
- accessibility snapshots
- patch diffs
- reports

Bind artifacts to `trace_id`, `commit_sha`, `source_repo`, `tool_version`, and `created_at`.

## React And Ant Design Impact Analysis

When frontend source is available, use AST parsing instead of regex for TSX/JSX.

Extract:

- routes and page components
- AntD component usage: Button, Form, Input, Select, Table, Modal, Dropdown, DatePicker
- `data-testid`, `aria-label`, `role`, `name`, `placeholder`, labels, button text
- event handlers: `onClick`, `onChange`, `onFinish`, `onSubmit`
- dialog and portal scope

Use this to map:

```text
commit -> changed file -> component -> route -> element -> locator -> BDD step -> scenario
```

## Output Artifacts

For MVP work, produce or update:

```text
config/project_binding.yaml
artifacts/{trace_id}/locator_inventory.json
artifacts/{trace_id}/page_snapshot.html
artifacts/{trace_id}/accessibility_snapshot.json
artifacts/{trace_id}/locator_candidates.json
artifacts/{trace_id}/validation_results.json
artifacts/{trace_id}/java_patch.diff
artifacts/{trace_id}/rerun_result.json
artifacts/{trace_id}/healing_report.md
```

When asked for planning, write phases in this order:

1. baseline binding
2. locator self-healing MVP
3. page element scanning and knowledge base
4. frontend AST/Repo-Map impact analysis
5. PR-level impacted test selection
6. Java patch governance
7. story-card test generation
8. enterprise audit and metrics

When asked for harness improvements, include:

1. workflow routing
2. progress and handoff files
3. browser/runtime verification
4. eval and CI gates
5. knowledge garbage collection
6. completion status protocol
7. security and privacy stop-gates

## References

Read `references/research-notes.md` when current external tool choices, source links, or implementation tradeoffs are needed.
