# Python UI 自愈测试 MVP 伪代码

## 1. MVP 目标

这个 MVP 先解决最小闭环：

```text
Java + BDD + Playwright 测试失败
  -> 解析失败 locator 和 step
  -> 采集当前页面 DOM / a11y / 截图
  -> 从知识库召回历史元素画像
  -> LLM 生成候选 locator
  -> Playwright 验证候选
  -> 修改 Java locator
  -> 重跑失败场景
  -> 记录自愈事件
```

MVP 不追求一开始就自动覆盖所有页面。先选择一个业务页面、十几个失败样本，把“失败到补丁到重跑”的链路打通。

## 2. 目录结构示例

```text
self_healing_mvp/
  app.py
  agents/
    graph.py
    failure_analyzer.py
    element_retriever.py
    locator_generator.py
    runtime_validator.py
    java_patch_agent.py
    report_agent.py
  scanners/
    java_bdd_scanner.py
    frontend_scanner.py
  storage/
    repository.py
    vector_store.py
    object_store.py
  runners/
    java_test_runner.py
    playwright_probe.py
  models/
    schemas.py
  prompts/
    locator_healing.md
    java_patch.md
```

## 3. 核心数据模型

```python
from pydantic import BaseModel
from typing import Literal, Optional


class FailedStep(BaseModel):
    run_id: str
    feature_file: str
    scenario_name: str
    step_text: str
    step_definition_file: str
    step_definition_line: int
    page_object_file: Optional[str]
    locator_line: Optional[int]
    old_locator: str
    action: Literal["click", "fill", "select", "assert_visible", "assert_text"]
    error_type: str
    error_message: str
    screenshot_uri: Optional[str]
    trace_uri: Optional[str]
    current_url: Optional[str]


class UiElementProfile(BaseModel):
    element_uid: str
    page_id: str
    route: str
    business_name: str
    intent: str
    historical_locators: list[str]
    role: Optional[str]
    accessible_name: Optional[str]
    label: Optional[str]
    placeholder: Optional[str]
    nearby_texts: list[str]
    component_path: Optional[str]
    screenshot_uri: Optional[str]


class PageSnapshot(BaseModel):
    url: str
    dom_text: str
    accessibility_tree: str
    screenshot_uri: str
    html_uri: str


class LocatorCandidate(BaseModel):
    locator_type: Literal["testId", "role", "label", "placeholder", "text", "css", "xpath"]
    playwright_java_code: str
    playwright_python_probe: str
    reason: str
    confidence: float
    risk: Literal["low", "medium", "high"]


class ValidationResult(BaseModel):
    candidate: LocatorCandidate
    is_valid: bool
    count: int
    visible: bool
    enabled: Optional[bool]
    editable: Optional[bool]
    action_replay_passed: bool
    score: float
    failure_reason: Optional[str]


class HealingPatch(BaseModel):
    file_path: str
    old_code: str
    new_code: str
    diff: str
    validation: ValidationResult
    requires_review: bool
```

## 4. LangGraph 编排伪代码

```python
from langgraph.graph import StateGraph, END


class HealingState(dict):
    """
    state fields:
      raw_failure_log
      failed_step
      element_profile
      page_snapshot
      locator_candidates
      validation_results
      selected_candidate
      patch
      rerun_result
      report
    """


def build_graph():
    graph = StateGraph(HealingState)

    graph.add_node("analyze_failure", analyze_failure)
    graph.add_node("retrieve_element_profile", retrieve_element_profile)
    graph.add_node("collect_page_snapshot", collect_page_snapshot)
    graph.add_node("generate_locator_candidates", generate_locator_candidates)
    graph.add_node("validate_candidates", validate_candidates)
    graph.add_node("patch_java_code", patch_java_code)
    graph.add_node("rerun_failed_scenario", rerun_failed_scenario)
    graph.add_node("write_healing_report", write_healing_report)
    graph.add_node("manual_triage", manual_triage)

    graph.set_entry_point("analyze_failure")
    graph.add_edge("analyze_failure", "retrieve_element_profile")
    graph.add_edge("retrieve_element_profile", "collect_page_snapshot")
    graph.add_edge("collect_page_snapshot", "generate_locator_candidates")
    graph.add_edge("generate_locator_candidates", "validate_candidates")

    graph.add_conditional_edges(
        "validate_candidates",
        should_patch,
        {
            "patch": "patch_java_code",
            "triage": "manual_triage",
        },
    )

    graph.add_edge("patch_java_code", "rerun_failed_scenario")

    graph.add_conditional_edges(
        "rerun_failed_scenario",
        rerun_passed,
        {
            "passed": "write_healing_report",
            "failed": "manual_triage",
        },
    )

    graph.add_edge("write_healing_report", END)
    graph.add_edge("manual_triage", END)

    return graph.compile()
```

## 5. 失败分析节点

```python
def analyze_failure(state: HealingState) -> HealingState:
    log = state["raw_failure_log"]

    failed_step = parse_cucumber_and_playwright_log(log)

    if failed_step.error_type not in {
        "LOCATOR_NOT_FOUND",
        "LOCATOR_NOT_UNIQUE",
        "ELEMENT_NOT_VISIBLE",
        "ELEMENT_NOT_ENABLED",
        "ELEMENT_OBSCURED",
    }:
        state["non_healable_reason"] = "Failure is not a locator-level failure"
        return state

    state["failed_step"] = failed_step
    return state


def parse_cucumber_and_playwright_log(log: str) -> FailedStep:
    """
    MVP 可以先做规则解析：
    - feature 文件和 scenario 从 Cucumber 输出中提取
    - Java 文件和行号从 stack trace 提取
    - old locator 从 Playwright timeout message 提取
    - action 从失败方法名或 step 文本推断

    后续再用 Java AST 和 LLM 增强。
    """
    return FailedStep(
        run_id="run-001",
        feature_file="src/test/resources/features/Login.feature",
        scenario_name="User login successfully",
        step_text="When user clicks login button",
        step_definition_file="src/test/java/steps/LoginSteps.java",
        step_definition_line=42,
        page_object_file="src/test/java/pages/LoginPage.java",
        locator_line=21,
        old_locator='page.locator("#loginBtn")',
        action="click",
        error_type="LOCATOR_NOT_FOUND",
        error_message="Timeout 30000ms exceeded",
        screenshot_uri="s3://ui-kb/failures/run-001/login.png",
        trace_uri="s3://ui-kb/failures/run-001/trace.zip",
        current_url="https://test.example.com/login",
    )
```

## 6. 元素画像召回节点

```python
def retrieve_element_profile(state: HealingState) -> HealingState:
    failed_step: FailedStep = state["failed_step"]

    # 1. 结构化精确召回
    profile = repository.find_element_by_locator(
        locator=failed_step.old_locator,
        file=failed_step.page_object_file,
        line=failed_step.locator_line,
    )

    # 2. 如果精确召回失败，使用语义召回
    if profile is None:
        query = f"""
        scenario: {failed_step.scenario_name}
        step: {failed_step.step_text}
        action: {failed_step.action}
        old locator: {failed_step.old_locator}
        error: {failed_step.error_message}
        """
        profile = vector_store.search_element_profile(query, top_k=5)[0]

    state["element_profile"] = profile
    return state
```

## 7. 当前页面采集节点

```python
from playwright.sync_api import sync_playwright


def collect_page_snapshot(state: HealingState) -> HealingState:
    failed_step: FailedStep = state["failed_step"]

    # MVP 做法：使用失败 trace 或重新跑到失败前状态。
    # 更稳的做法是让 Java 测试 runner 支持在失败 step 前暂停并暴露 browser ws endpoint。
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()

        restore_test_session(page, failed_step)
        navigate_to_failure_state(page, failed_step)

        html = page.content()
        dom_text = page.locator("body").inner_text(timeout=5000)
        screenshot_path = object_store.save_screenshot(page.screenshot(full_page=True))

        # Playwright Python 没有和 Node 完全一致的 public accessibility API 时，
        # MVP 可通过 CDP 或 Playwright MCP 获取 a11y snapshot。
        a11y_tree = capture_accessibility_tree(page)

        html_uri = object_store.save_text(html, suffix=".html")

        browser.close()

    state["page_snapshot"] = PageSnapshot(
        url=failed_step.current_url,
        dom_text=dom_text,
        accessibility_tree=a11y_tree,
        screenshot_uri=screenshot_path,
        html_uri=html_uri,
    )
    return state
```

## 8. 候选 locator 生成节点

```python
def generate_locator_candidates(state: HealingState) -> HealingState:
    failed_step: FailedStep = state["failed_step"]
    profile: UiElementProfile = state["element_profile"]
    snapshot: PageSnapshot = state["page_snapshot"]

    prompt_input = {
        "failed_step": failed_step.model_dump(),
        "historical_element_profile": profile.model_dump(),
        "current_page": snapshot.model_dump(),
        "locator_policy": [
            "Prefer data-testid if semantically stable",
            "Prefer role + accessible name for user-facing elements",
            "Use label or placeholder for form fields",
            "Avoid absolute xpath",
            "Avoid dynamic class names",
            "Return Java Playwright code and Python probe locator",
        ],
    }

    llm_response = gpt_52_json(
        system="You are a senior UI test self-healing agent.",
        prompt_template="prompts/locator_healing.md",
        data=prompt_input,
        response_schema=list[LocatorCandidate],
    )

    state["locator_candidates"] = llm_response
    return state
```

示例 LLM 输出：

```json
[
  {
    "locator_type": "role",
    "playwright_java_code": "page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName(\"Sign in\"))",
    "playwright_python_probe": "page.get_by_role(\"button\", name=\"Sign in\")",
    "reason": "当前页面存在唯一可见的 Sign in button，语义与登录提交一致",
    "confidence": 0.92,
    "risk": "low"
  },
  {
    "locator_type": "testId",
    "playwright_java_code": "page.getByTestId(\"login-submit-button\")",
    "playwright_python_probe": "page.get_by_test_id(\"login-submit-button\")",
    "reason": "当前 DOM 存在稳定 data-testid",
    "confidence": 0.95,
    "risk": "low"
  }
]
```

## 9. 候选验证节点

```python
def validate_candidates(state: HealingState) -> HealingState:
    failed_step: FailedStep = state["failed_step"]
    candidates: list[LocatorCandidate] = state["locator_candidates"]

    results = []

    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()

        restore_test_session(page, failed_step)
        navigate_to_failure_state(page, failed_step)

        for candidate in candidates:
            result = validate_one_candidate(page, candidate, failed_step.action)
            results.append(result)

        browser.close()

    valid_results = [r for r in results if r.is_valid]
    valid_results.sort(key=lambda r: r.score, reverse=True)

    state["validation_results"] = results

    if valid_results:
        state["selected_candidate"] = valid_results[0].candidate
    else:
        state["selected_candidate"] = None

    return state


def validate_one_candidate(page, candidate: LocatorCandidate, action: str) -> ValidationResult:
    locator = eval_python_probe_safely(page, candidate.playwright_python_probe)

    count = locator.count()
    if count != 1:
        return ValidationResult(
            candidate=candidate,
            is_valid=False,
            count=count,
            visible=False,
            enabled=None,
            editable=None,
            action_replay_passed=False,
            score=0,
            failure_reason=f"Locator matched {count} elements",
        )

    visible = locator.is_visible()
    if not visible:
        return invalid(candidate, count, "Element is not visible")

    enabled = None
    editable = None
    action_ok = False

    try:
        if action == "click":
            enabled = locator.is_enabled()
            if enabled:
                locator.click(trial=True)
                action_ok = True

        elif action == "fill":
            editable = locator.is_editable()
            if editable:
                locator.fill("self-healing-probe", timeout=2000)
                action_ok = True

        elif action == "assert_visible":
            action_ok = visible

        elif action == "assert_text":
            action_ok = visible and bool(locator.inner_text(timeout=2000))

    except Exception as exc:
        return invalid(candidate, count, f"Action replay failed: {exc}")

    score = calculate_score(
        candidate_confidence=candidate.confidence,
        unique=count == 1,
        visible=visible,
        enabled=enabled,
        editable=editable,
        action_ok=action_ok,
        locator_type=candidate.locator_type,
    )

    return ValidationResult(
        candidate=candidate,
        is_valid=action_ok and score >= 0.75,
        count=count,
        visible=visible,
        enabled=enabled,
        editable=editable,
        action_replay_passed=action_ok,
        score=score,
        failure_reason=None,
    )
```

安全注意：真实实现不建议直接 `eval` LLM 输出。MVP 里也应把 `playwright_python_probe` 限制为一个 JSON DSL，再由代码转换为 Playwright 调用。

推荐 DSL：

```json
{
  "method": "get_by_role",
  "args": ["button"],
  "kwargs": {"name": "Sign in"}
}
```

## 10. 是否进入补丁节点

```python
def should_patch(state: HealingState) -> str:
    candidate = state.get("selected_candidate")
    if candidate is None:
        return "triage"

    best = max(
        [r for r in state["validation_results"] if r.is_valid],
        key=lambda r: r.score,
    )

    if best.score < 0.75:
        return "triage"

    if candidate.risk == "high":
        return "triage"

    return "patch"
```

## 11. Java 代码补丁节点

```python
def patch_java_code(state: HealingState) -> HealingState:
    failed_step: FailedStep = state["failed_step"]
    candidate: LocatorCandidate = state["selected_candidate"]

    source = read_file(failed_step.page_object_file)

    # MVP 可以先按 old locator 精确替换。
    # 生产级需要 JavaParser/tree-sitter 定位 AST 节点，避免误替换。
    patch_plan = gpt_52_json(
        system="You are a Java Playwright code patch agent.",
        prompt_template="prompts/java_patch.md",
        data={
            "file": failed_step.page_object_file,
            "old_locator": failed_step.old_locator,
            "new_locator": candidate.playwright_java_code,
            "source": source,
            "rules": [
                "Only update locator code",
                "Do not change business flow",
                "Keep current style",
                "Prefer replacing constants or Page Object methods",
            ],
        },
        response_schema=dict,
    )

    new_source = apply_ast_patch_or_safe_replace(source, patch_plan)
    diff = create_unified_diff(source, new_source, failed_step.page_object_file)

    write_file(failed_step.page_object_file, new_source)

    state["patch"] = HealingPatch(
        file_path=failed_step.page_object_file,
        old_code=patch_plan["old_code"],
        new_code=patch_plan["new_code"],
        diff=diff,
        validation=max(
            [r for r in state["validation_results"] if r.is_valid],
            key=lambda r: r.score,
        ),
        requires_review=patch_plan.get("requires_review", True),
    )

    return state
```

## 12. 重跑失败场景

```python
def rerun_failed_scenario(state: HealingState) -> HealingState:
    failed_step: FailedStep = state["failed_step"]

    result = java_test_runner.run_single_scenario(
        feature_file=failed_step.feature_file,
        scenario_name=failed_step.scenario_name,
        env={
            "HEALING_MODE": "verify_patch",
            "RUN_ID": failed_step.run_id,
        },
    )

    state["rerun_result"] = result
    return state


def rerun_passed(state: HealingState) -> str:
    result = state["rerun_result"]
    return "passed" if result.passed else "failed"
```

## 13. 报告与知识库更新

```python
def write_healing_report(state: HealingState) -> HealingState:
    failed_step = state["failed_step"]
    patch = state["patch"]
    rerun = state["rerun_result"]

    healing_event = {
        "run_id": failed_step.run_id,
        "scenario": failed_step.scenario_name,
        "step": failed_step.step_text,
        "old_locator": failed_step.old_locator,
        "new_locator": patch.new_code,
        "score": patch.validation.score,
        "diff": patch.diff,
        "rerun_passed": rerun.passed,
        "requires_review": patch.requires_review,
    }

    repository.save_healing_event(healing_event)
    vector_store.upsert_failure_case(healing_event)

    report_path = report_agent.render_markdown(healing_event)
    state["report"] = report_path

    return state
```

示例报告：

```markdown
# UI Self-Healing Report

- Scenario: User login successfully
- Step: When user clicks login button
- Failure: LOCATOR_NOT_FOUND
- Old locator: page.locator("#loginBtn")
- New locator: page.getByTestId("login-submit-button")
- Validation: unique=true, visible=true, click trial=true
- Score: 0.94
- Rerun: passed
- Review: required
```

## 14. Commit 影响分析 MVP 伪代码

```python
def on_frontend_commit(commit_sha: str):
    diff = git_client.get_diff(commit_sha)

    changed_components = frontend_scanner.extract_changed_components(diff)

    impacted_elements = repository.find_elements_by_components(changed_components)

    impacted_scenarios = graph_repository.find_scenarios_by_elements(
        impacted_elements
    )

    risk_report = []
    for element in impacted_elements:
        risk = calculate_locator_risk(element, diff)
        risk_report.append((element, risk))

    selected_tests = select_minimum_regression_set(
        impacted_scenarios,
        risk_report,
    )

    result = java_test_runner.run_scenarios(selected_tests)

    if result.has_locator_failures():
        for failure in result.locator_failures:
            healing_graph.invoke({"raw_failure_log": failure.raw_log})

    repository.save_commit_impact_report(
        commit_sha=commit_sha,
        changed_components=changed_components,
        impacted_elements=impacted_elements,
        impacted_scenarios=impacted_scenarios,
        selected_tests=selected_tests,
    )
```

## 15. 新故事卡生成 UI 自动化的伪代码

```python
def generate_tests_for_story(story_card):
    requirement_doc = normalize_story_card(story_card)

    frontend_context = frontend_scanner.scan_new_or_changed_pages(
        story_card.related_commits
    )

    runtime_pages = playwright_probe.explore_pages(
        routes=frontend_context.routes,
        seed_user=story_card.test_user,
    )

    new_elements = element_builder.build_profiles(
        frontend_context=frontend_context,
        runtime_pages=runtime_pages,
    )

    similar_scenarios = vector_store.search_bdd_steps(
        query=requirement_doc.acceptance_criteria,
        top_k=10,
    )

    bdd_plan = gpt_52_json(
        system="You are a senior BDD test designer.",
        data={
            "requirement": requirement_doc,
            "new_elements": [e.model_dump() for e in new_elements],
            "similar_scenarios": similar_scenarios,
            "rules": [
                "Generate business-readable Gherkin",
                "Reuse existing step definitions when possible",
                "Cover happy path, validation, permission, and error path",
                "Do not invent UI elements not present in runtime snapshot",
            ],
        },
        response_schema=dict,
    )

    java_steps = gpt_52_json(
        system="You are a Java Playwright BDD automation engineer.",
        data={
            "bdd_plan": bdd_plan,
            "element_profiles": [e.model_dump() for e in new_elements],
            "existing_step_definitions": repository.find_reusable_steps(bdd_plan),
            "coding_rules": repository.get_project_coding_rules(),
        },
        response_schema=dict,
    )

    create_draft_pr(
        files={
            "feature": bdd_plan["feature_file"],
            "java": java_steps["files"],
        },
        label="ai-generated-ui-test",
        requires_review=True,
    )
```

## 16. MVP 运行入口

```python
def main():
    graph = build_graph()

    # 模拟从 CI 读取失败日志
    raw_failure_log = read_file("artifacts/latest-cucumber-failure.log")

    result = graph.invoke({
        "raw_failure_log": raw_failure_log,
    })

    print("Healing finished")
    print("Report:", result.get("report"))
    print("Patch:", result.get("patch"))


if __name__ == "__main__":
    main()
```

## 17. MVP 验收标准

MVP 达标条件：

- 能从 Java BDD 失败日志中识别失败 step、旧 locator 和代码位置。
- 能采集当前页面 DOM、a11y tree 和截图。
- 能生成不少于 3 个候选 locator。
- 能用 Playwright 验证候选 locator 是否唯一、可见、可执行动作。
- 能将最佳候选转换为 Java Playwright locator。
- 能生成最小代码 diff。
- 能重跑失败场景并记录结果。
- 能输出自愈报告。

不建议在 MVP 阶段自动合并代码。建议自动创建修复 PR 或报告，由测试负责人 review。

## 18. 结合 UI-Root 的 MVP 实施切片

`UI-Root.html` 的完整链路较大，MVP 不应一次实现所有 Agent。建议按“可验证闭环优先”的原则，把六层压缩成一个可跑通的最小链路。

### 18.1 MVP 必做节点

| UI-Root 节点 | MVP 是否实现 | 最小实现方式 |
| --- | --- | --- |
| 前端代码提交 Webhook | 实现基础版 | 本地读取指定 commit diff 或 PR diff 文件 |
| 关联初始前端和自动化项目 | 实现基础版 | 手工配置 `project_binding.yaml`，导入 1 个页面和 10-50 个场景 |
| Orchestrator | 实现 | LangGraph StateGraph + SQLite/PostgreSQL checkpoint |
| 分支同步管理器 | 实现基础版 | 创建本地 `auto-heal-{run_id}` 分支，支持 apply/rollback |
| Change Analyzer | 实现基础版 | 只解析 TSX 中 AntD Button/Form/Input/Select/Table/Modal |
| Impact Predictor | 实现基础版 | 基于 route_map 和手工映射选择最小回归集合 |
| Locator Healer | 实现 | 生成 3 个以上候选 locator，并用 Playwright 验证 |
| Patch Generator | 实现 | 只修改 Java Page Object locator，不改业务步骤 |
| Playwright Runner | 实现 | Maven/Gradle 单场景重跑，归档 trace/screenshot |
| 人工合并 | 实现为报告 | 生成 PR diff 和 Markdown，不自动 merge |
| 结果判定路由 | 实现 | 分类 L1/L2/L3，只有 L1 进入自动 patch |
| 运行标记表 | 实现基础版 | 保存 run_id、状态、耗时、分支、结论 |
| 修复记录数据库 | 实现基础版 | 保存 failure_diagnosis、patch、rerun_result |
| 知识库存储 | 实现基础版 | JSON artifact + SQLite/PostgreSQL，向量库可延后 |

### 18.2 建议新增配置文件

```text
self_healing_mvp/
  config/
    project_binding.yaml
    locator_policy.yaml
    repair_policy.yaml
    env_policy.yaml
```

`project_binding.yaml` 示例：

```yaml
frontend_repo: ../frontend
automation_repo: ../e2e-java
base_url: https://test.example.com
default_branch: main
test_command: mvn test -Dcucumber.filter.tags="@trade-search"
routes:
  trade-search:
    path: /trade/search
    source_files:
      - src/pages/trade/SearchPage.tsx
    feature_files:
      - src/test/resources/features/TradeSearch.feature
```

### 18.3 LangGraph 状态补充

建议把当前伪代码中的 `HealingState` 扩展为同时支持 PR 影响分析和失败后自愈：

```python
class HealingState(dict):
    """
    trace_id
    commit_sha
    project_binding
    sandbox_branch
    diff_summary
    changed_components
    impacted_routes
    impacted_scenarios
    raw_failure_log
    failed_step
    page_snapshot
    element_profile
    locator_candidates
    validation_results
    selected_candidate
    failure_diagnosis
    patch
    rerun_result
    review_level
    report
    """
```

### 18.4 MVP 流程细化

```text
init_project_binding
  -> create_sandbox_branch
  -> analyze_frontend_diff
  -> predict_impacted_tests
  -> run_impacted_tests
  -> classify_result
  -> if locator failure: heal_locator
  -> validate_locator_candidates
  -> generate_java_patch
  -> apply_patch_on_sandbox
  -> rerun_failed_scenario
  -> write_healing_report
  -> persist_knowledge_artifacts
```

### 18.5 失败分类最小规则

```python
def classify_failure(error_message: str, trace_summary: dict) -> str:
    if "Timeout" in error_message and "locator" in error_message:
        return "LOCATOR_NOT_FOUND"
    if "strict mode violation" in error_message:
        return "LOCATOR_NOT_UNIQUE"
    if "not visible" in error_message:
        return "ELEMENT_NOT_VISIBLE"
    if trace_summary.get("api_errors"):
        return "NETWORK_OR_BACKEND_FAILURE"
    if trace_summary.get("console_errors"):
        return "ENV_OR_FRONTEND_RUNTIME_FAILURE"
    return "MANUAL_TRIAGE_REQUIRED"
```

只有 `LOCATOR_NOT_FOUND`、`LOCATOR_NOT_UNIQUE`、`ELEMENT_NOT_VISIBLE` 在 MVP 自动修复范围内。

### 18.6 输出 artifact 清单

每次运行至少输出以下文件，便于后续接入正式知识库：

```text
artifacts/{trace_id}/
  diff_summary.json
  impacted_tests.json
  failure_diagnosis.json
  page_snapshot.html
  accessibility_snapshot.json
  locator_candidates.json
  validation_results.json
  java_patch.diff
  rerun_result.json
  healing_report.md
```

### 18.7 MVP 演示路径

推荐演示用例选一个 AntD 表单页面：

1. 历史 Java BDD 用例使用 `getByPlaceholder("Search counterparty")`。
2. 前端 PR 修改 placeholder 或把 `Input` 改为 `Select`，触发影响分析。
3. Runner 执行受影响场景并失败。
4. Locator Healer 基于当前 a11y/DOM 找到 `data-testid` 或 `getByRole/getByLabel` 候选。
5. Runtime Validator 验证唯一、可见、可交互。
6. Patch Generator 修改 Java Page Object。
7. 重跑单场景通过。
8. 输出报告和补丁，进入人工 review。

### 18.8 MVP 不做但要留接口

- 不做全量故事卡自动生成，但保留 `story_requirement` 输入模型。
- 不做自动合并主干，但保留 `create_pr` 接口。
- 不做跨项目共享知识库，但 artifact 命名必须带 `source_repo`。
- 不做视觉模型兜底，但保存截图和 element crop，方便后续扩展。
- 不做复杂 Java AST patch，先以 Page Object 方法级安全替换为主，接口命名保留 `java_ast_patcher`。
