# V1 数据库 DDL（PostgreSQL）

更新时间：2026-04-28

## 1. 目标

定义 V1 最小可用数据模型，满足：

1. webhook 事件入库与幂等
2. 多 Job 并行 run 追踪
3. 失败分类与决策记录
4. 分支/MR/revert 审计
5. 工件证据可追溯

## 2. DDL

```sql
-- Optional extension for UUID generation
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE IF NOT EXISTS project_binding (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_key         TEXT NOT NULL UNIQUE,
  display_name        TEXT NOT NULL,
  gitlab_project_id   TEXT NOT NULL,
  gitlab_default_ref  TEXT NOT NULL DEFAULT 'main',
  jenkins_base_url    TEXT NOT NULL,
  timezone            TEXT NOT NULL DEFAULT 'Asia/Shanghai',
  enable_heal_write   BOOLEAN NOT NULL DEFAULT FALSE,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE IF NOT EXISTS job_binding (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id          UUID NOT NULL REFERENCES project_binding(id),
  job_key             TEXT NOT NULL,
  job_name            TEXT NOT NULL,
  trigger_mode        TEXT NOT NULL DEFAULT 'webhook',
  parallel_limit      INT  NOT NULL DEFAULT 3,
  is_active           BOOLEAN NOT NULL DEFAULT TRUE,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (project_id, job_key)
);

CREATE TABLE IF NOT EXISTS ci_event (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_id            TEXT NOT NULL UNIQUE,
  project_id          UUID NOT NULL REFERENCES project_binding(id),
  source              TEXT NOT NULL DEFAULT 'jenkins_webhook',
  payload_hash        TEXT NOT NULL,
  idem_key            TEXT,
  raw_payload         JSONB NOT NULL,
  received_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  status              TEXT NOT NULL DEFAULT 'RECEIVED',
  UNIQUE (project_id, payload_hash)
);

CREATE INDEX IF NOT EXISTS idx_ci_event_project_received
ON ci_event(project_id, received_at DESC);

CREATE TABLE IF NOT EXISTS ci_run (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              TEXT NOT NULL UNIQUE,
  event_id            UUID NOT NULL REFERENCES ci_event(id),
  project_id          UUID NOT NULL REFERENCES project_binding(id),
  job_id              UUID NOT NULL REFERENCES job_binding(id),
  job_name            TEXT NOT NULL,
  build_number        BIGINT,
  build_url           TEXT,
  branch              TEXT,
  commit_sha          TEXT,
  status              TEXT NOT NULL DEFAULT 'RECEIVED',
  aggregate_group_id  TEXT,
  started_at          TIMESTAMPTZ,
  finished_at         TIMESTAMPTZ,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_ci_run_event
ON ci_run(event_id);

CREATE INDEX IF NOT EXISTS idx_ci_run_project_status
ON ci_run(project_id, status);

CREATE TABLE IF NOT EXISTS artifact_ref (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID NOT NULL REFERENCES ci_run(id),
  artifact_type       TEXT NOT NULL, -- screenshot|trace|junit|log|html
  original_path       TEXT,
  fetched_uri         TEXT,
  canonical_name      TEXT,
  fetch_status        TEXT NOT NULL DEFAULT 'PENDING',
  size_bytes          BIGINT,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_artifact_run
ON artifact_ref(run_id);

CREATE TABLE IF NOT EXISTS failure_case (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID NOT NULL REFERENCES ci_run(id),
  scenario_name       TEXT,
  step_text           TEXT,
  page_object_file    TEXT,
  locator_line        INT,
  old_locator         TEXT,
  error_type          TEXT NOT NULL,
  error_message       TEXT,
  stack_digest        TEXT,
  parser_version      TEXT,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_failure_run
ON failure_case(run_id);

CREATE TABLE IF NOT EXISTS decision_record (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID NOT NULL REFERENCES ci_run(id),
  decision            TEXT NOT NULL, -- NO_HEAL|HEAL_CANDIDATE|NEED_HUMAN
  reason_code         TEXT NOT NULL,
  reason_detail       TEXT,
  decided_by          TEXT NOT NULL DEFAULT 'policy_engine',
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX IF NOT EXISTS uq_decision_run
ON decision_record(run_id);

CREATE TABLE IF NOT EXISTS healing_attempt (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID NOT NULL REFERENCES ci_run(id),
  source_branch       TEXT,
  heal_branch         TEXT,
  base_sha            TEXT,
  patch_commit_sha    TEXT,
  mr_iid              TEXT,
  mr_title            TEXT,
  ci_rerun_status     TEXT,
  revert_status       TEXT, -- SUCCESS|FAILED|CONFLICT|SKIPPED
  revert_commit_sha   TEXT,
  status              TEXT NOT NULL DEFAULT 'CREATED',
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX IF NOT EXISTS uq_healing_run
ON healing_attempt(run_id);

CREATE TABLE IF NOT EXISTS git_action_log (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id              UUID REFERENCES ci_run(id),
  action_type         TEXT NOT NULL, -- create_branch|create_commit|create_mr|revert_commit|query_mr
  request_payload     JSONB,
  response_payload    JSONB,
  success             BOOLEAN NOT NULL,
  error_message       TEXT,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_git_action_run
ON git_action_log(run_id);

CREATE TABLE IF NOT EXISTS audit_log (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id          TEXT,
  actor               TEXT NOT NULL DEFAULT 'system',
  action              TEXT NOT NULL,
  target_type         TEXT,
  target_id           TEXT,
  result              TEXT NOT NULL, -- SUCCESS|FAIL
  detail              JSONB,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_audit_created
ON audit_log(created_at DESC);
```

## 3. 状态字段建议枚举

1. `ci_run.status`：`RECEIVED`、`ENRICHING_CONTEXT`、`FETCHING_ARTIFACTS`、`PARSING_FAILURE`、`CLASSIFIED`、`NO_HEAL`、`HEAL_CANDIDATE`、`CI_RERUN_TRACKING`、`DONE`、`FAILED`
2. `healing_attempt.status`：`CREATED`、`BRANCH_CREATED`、`PATCH_COMMITTED`、`MR_CREATED`、`RERUN_FAILED`、`REVERTED`、`REVERT_CONFLICT`、`CLOSED`

## 4. 审计必填追溯字段

以下字段在业务对象中必须可追溯：

1. `event_id`
2. `run_id`
3. `job_name`
4. `build_number`
5. `branch`
6. `commit_sha`
7. `timezone`
