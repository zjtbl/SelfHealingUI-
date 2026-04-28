# V1 LangGraph 状态机设计

更新时间：2026-04-28  
时区：`Asia/Shanghai`

## 1. 目标

将“Jenkins 回调 -> 失败分析 -> 决策 ->（可选）自愈分支流”固化成可追踪状态机。

V1 默认启用：失败分析闭环。  
V1.5 通过开关启用：locator 修复写分支闭环。

## 2. 状态对象（建议）

```python
from typing import Optional, Literal, TypedDict, Any

Decision = Literal["NO_HEAL", "HEAL_CANDIDATE", "NEED_HUMAN"]
RunStatus = Literal[
    "RECEIVED",
    "ENRICHING_CONTEXT",
    "FETCHING_ARTIFACTS",
    "PARSING_FAILURE",
    "CLASSIFIED",
    "NO_HEAL",
    "HEAL_CANDIDATE",
    "HEALING_FLOW",
    "CI_RERUN_TRACKING",
    "REVERTED",
    "DONE",
    "FAILED",
]

class RunState(TypedDict, total=False):
    request_id: str
    event_id: str
    run_id: str
    project_id: str
    job_id: str
    job_name: str
    build_number: int
    build_url: str
    branch: str
    commit_sha: str
    timezone: str
    raw_payload: dict[str, Any]
    artifacts: list[dict[str, Any]]
    parsed_failure: dict[str, Any]
    decision: Decision
    decision_reason: str
    heal_write_enabled: bool
    heal_branch: Optional[str]
    patch_commit_sha: Optional[str]
    mr_iid: Optional[str]
    rerun_status: Optional[str]
    revert_status: Optional[str]
    status: RunStatus
    error: Optional[str]
```

## 3. 节点定义

1. `receive_webhook`
2. `enrich_context`
3. `fetch_artifacts`
4. `parse_failure`
5. `classify_failure`
6. `route_decision`
7. `record_no_heal`
8. `start_healing_flow`（V1.5）
9. `create_heal_branch`（V1.5）
10. `commit_patch`（V1.5）
11. `create_draft_mr`（V1.5）
12. `trigger_rerun_ci`（V1.5）
13. `check_rerun_result`（V1.5）
14. `revert_commit`（V1.5）
15. `mark_revert_conflict`
16. `finalize_done`
17. `finalize_failed`

## 4. 条件路由

1. webhook 字段完整：直接进入 `fetch_artifacts`
2. 字段缺失：进入 `enrich_context`，通过 `build_url` 补全
3. 解析后非 locator 类：`NO_HEAL`
4. 解析后 locator 类：`HEAL_CANDIDATE`
5. `HEAL_CANDIDATE` + `heal_write_enabled=false`：仅记录，不写代码
6. `HEAL_CANDIDATE` + `heal_write_enabled=true`：进入 V1.5 分支流
7. 分支 CI 失败：执行 `revert_commit`
8. `revert_commit` 冲突：`mark_revert_conflict` 并停止

## 5. 状态迁移图（文本）

```text
RECEIVED
  -> ENRICHING_CONTEXT
  -> FETCHING_ARTIFACTS
  -> PARSING_FAILURE
  -> CLASSIFIED
  -> (NO_HEAL | HEAL_CANDIDATE | NEED_HUMAN)

NO_HEAL
  -> DONE

HEAL_CANDIDATE (heal_write_enabled=false)
  -> DONE

HEAL_CANDIDATE (heal_write_enabled=true)
  -> HEALING_FLOW
  -> CI_RERUN_TRACKING
  -> (DONE | REVERTED | FAILED)
```

## 6. 伪代码（LangGraph）

```python
from langgraph.graph import StateGraph, END

def build_graph():
    g = StateGraph(RunState)

    g.add_node("receive_webhook", receive_webhook)
    g.add_node("enrich_context", enrich_context)
    g.add_node("fetch_artifacts", fetch_artifacts)
    g.add_node("parse_failure", parse_failure)
    g.add_node("classify_failure", classify_failure)
    g.add_node("record_no_heal", record_no_heal)
    g.add_node("start_healing_flow", start_healing_flow)
    g.add_node("create_heal_branch", create_heal_branch)
    g.add_node("commit_patch", commit_patch)
    g.add_node("create_draft_mr", create_draft_mr)
    g.add_node("trigger_rerun_ci", trigger_rerun_ci)
    g.add_node("check_rerun_result", check_rerun_result)
    g.add_node("revert_commit", revert_commit)
    g.add_node("mark_revert_conflict", mark_revert_conflict)
    g.add_node("finalize_done", finalize_done)
    g.add_node("finalize_failed", finalize_failed)

    g.set_entry_point("receive_webhook")
    g.add_edge("receive_webhook", "enrich_context")
    g.add_edge("enrich_context", "fetch_artifacts")
    g.add_edge("fetch_artifacts", "parse_failure")
    g.add_edge("parse_failure", "classify_failure")

    g.add_conditional_edges(
        "classify_failure",
        route_decision,
        {
            "NO_HEAL": "record_no_heal",
            "HEAL_ANALYZE_ONLY": "finalize_done",
            "HEAL_WITH_WRITE": "start_healing_flow",
            "FAILED": "finalize_failed",
        },
    )

    g.add_edge("record_no_heal", "finalize_done")
    g.add_edge("start_healing_flow", "create_heal_branch")
    g.add_edge("create_heal_branch", "commit_patch")
    g.add_edge("commit_patch", "create_draft_mr")
    g.add_edge("create_draft_mr", "trigger_rerun_ci")
    g.add_edge("trigger_rerun_ci", "check_rerun_result")

    g.add_conditional_edges(
        "check_rerun_result",
        route_rerun,
        {
            "PASS": "finalize_done",
            "FAIL": "revert_commit",
        },
    )

    g.add_conditional_edges(
        "revert_commit",
        route_revert,
        {
            "REVERTED": "finalize_done",
            "CONFLICT": "mark_revert_conflict",
            "FAILED": "finalize_failed",
        },
    )

    g.add_edge("mark_revert_conflict", "finalize_failed")
    g.add_edge("finalize_done", END)
    g.add_edge("finalize_failed", END)

    return g.compile()
```

## 7. 并行策略

1. `event_id` 下可 fan-out 多个 `run_id`（每个 Job 一个 run）。
2. 每个 `run_id` 单独执行状态机实例。
3. 聚合器按 `event_id` 汇总：`ALL_SUCCESS`、`PARTIAL_FAIL`、`ALL_FAIL`。

## 8. 重试与失败策略

1. `fetch_artifacts` 可重试 3 次（指数退避）。
2. `parse_failure` 失败进入 `NEED_HUMAN`。
3. `create_branch/commit/mr/revert` 均写 `git_action_log`。
4. `revert` 冲突只打标，不做自动通知（后续版本补通知）。

## 9. V1 验收点

1. 能稳定完成 `RECEIVED -> CLASSIFIED -> NO_HEAL/HEAL_CANDIDATE`。
2. 分类结果和原因可在平台页面追溯。
3. 多 Job 并行 run 互不污染，聚合状态准确。
