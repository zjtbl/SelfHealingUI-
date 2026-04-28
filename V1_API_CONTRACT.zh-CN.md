# V1 API 契约（定稿）

更新时间：2026-04-28  
时区基线：`Asia/Shanghai`

## 1. 目标

定义独立 UI 自愈平台（控制面）在 V1 阶段的接口契约，覆盖：

1. Jenkins webhook 回调入口
2. 平台主动触发 Jenkins Job
3. 事件、运行、决策、MR 改动查询
4. 失败分析闭环（V1）
5. locator 自愈分支流接口预留（V1.5）

## 2. 统一约定

1. 认证：`Authorization: Bearer <token>`
2. 幂等键：`X-Idempotency-Key`（可选，建议）
3. 返回格式：`application/json`
4. 时间字段：ISO 8601，保存为 `timestamptz`
5. 错误结构：

```json
{
  "error_code": "INVALID_PAYLOAD",
  "message": "job_name is missing",
  "request_id": "req_01H..."
}
```

## 3. Jenkins Webhook

### 3.1 `POST /api/v1/webhooks/jenkins`

用途：接收 Jenkins 回调并启动失败分析流程。

请求体（宽松模式，字段不全也接受）：

```json
{
  "job_name": "e2e-ui-regression",
  "build_number": 1286,
  "build_url": "https://jenkins.example.com/job/e2e-ui-regression/1286/",
  "status": "FAILURE",
  "branch": "feature/login",
  "commit_sha": "3f8c9a...",
  "project_key": "payment-portal",
  "triggered_at": "2026-04-28T10:40:00+08:00",
  "raw_payload": {}
}
```

返回：

```json
{
  "event_id": "evt_20260428_0001",
  "accepted": true,
  "next_state": "RECEIVED"
}
```

行为约束：

1. 若关键字段缺失，平台先接收并落 raw payload。
2. 后续由 `mcp-jenkins` 通过 `build_url` 补全上下文。
3. 重复 webhook 由幂等逻辑去重，不重复执行流程。

## 4. 平台主动触发 Jenkins

### 4.1 `POST /api/v1/projects/{project_id}/jobs/{job_id}/trigger`

用途：平台主动触发 Jenkins Job。

请求体：

```json
{
  "branch": "main",
  "commit_sha": "3f8c9a...",
  "reason": "frontend_push_or_manual",
  "parameters": {
    "env": "staging",
    "suite": "smoke"
  }
}
```

返回：

```json
{
  "run_id": "run_20260428_100001",
  "queue_ref": "jenkins_queue_99181",
  "status": "TRIGGERED"
}
```

## 5. 事件与运行查询

### 5.1 `GET /api/v1/events/{event_id}`

返回该事件下的：

1. webhook 原始数据摘要
2. fan-out Job 列表
3. 聚合状态（`ALL_SUCCESS`/`PARTIAL_FAIL`/`ALL_FAIL`）
4. 决策摘要（`NO_HEAL`/`HEAL_CANDIDATE`/`NEED_HUMAN`）

### 5.2 `GET /api/v1/runs/{run_id}`

返回单次运行全链路：

1. 工件解析状态
2. 失败分类
3. 决策结果
4. 分支/MR/回退状态（若启用 V1.5）

## 6. MR 改动展示

### 6.1 `GET /api/v1/runs/{run_id}/mr-changes`

用途：平台页面查看自动创建 MR 的代码改动。

返回：

```json
{
  "mr_iid": 231,
  "source_branch": "ai/heal/payment-portal/20260428-104000/feature-login/3f8c9a",
  "target_branch": "main",
  "title": "[AI-HEAL][payment-portal] 2026-04-28 10:40 | feature/login | 3f8c9a",
  "changes": [
    {
      "file_path": "src/test/java/pages/LoginPage.java",
      "diff": "@@ ...",
      "is_locator_change": true
    }
  ]
}
```

## 7. 重试与回退

### 7.1 `POST /api/v1/runs/{run_id}/retry`

用途：人工触发该 run 重试。

### 7.2 `POST /api/v1/runs/{run_id}/revert`

用途：手动触发回退（`revert commit`）。

约束：

1. 回退冲突时返回 `REVERT_CONFLICT`。
2. 平台只打标失败，不执行额外自动修复。

## 8. 管理接口（项目与 Job）

### 8.1 `POST /api/v1/projects`

创建项目绑定（GitLab + Jenkins + 默认分支）。

### 8.2 `POST /api/v1/projects/{project_id}/jobs`

新增 Job 绑定。

### 8.3 `GET /api/v1/projects/{project_id}/jobs`

查看项目下 Job 列表与并发限制。

## 9. 命名规范（追溯）

1. 分支：`ai/heal/{projectKey}/{yyyyMMdd-HHmmss}/{srcBranch}/{shortSha}`
2. MR 标题：`[AI-HEAL][{projectKey}] {yyyy-MM-dd HH:mm} | {srcBranch} | {shortSha}`
3. 提交信息：`ai-heal(locator): {failureType} | {srcBranch} | {shortSha} | run={runId}`

## 10. MCP 内部接口约束（平台内部）

### 10.1 `mcp-jenkins`

1. `get_build_meta(build_url)`
2. `download_artifacts(build_url)`
3. `trigger_job(job_name, params)`
4. `get_build_status(job_name, build_number)`

### 10.2 `mcp-gitlab`

1. `create_branch(project, branch, ref=main)`
2. `create_commit(project, branch, actions[])`
3. `create_draft_mr(project, source, target, title)`
4. `get_mr_changes(project, mr_iid)`
5. `revert_commit(project, sha, branch)`

### 10.3 `mcp-parser`

1. `parse_cucumber_junit(artifacts)`
2. `classify_failure(parsed)`
3. `extract_locator_context(parsed, logs, traces)`

## 11. V1 非目标

1. 自动合并主干
2. 断言语义自动修复
3. 多存储后端（先 Jenkins 直连工件）
4. 通知系统（回退冲突仅打标）
