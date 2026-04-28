# V1 Database DDL

Updated: 2026-04-28

This is the English companion of [V1_DATABASE_DDL.zh-CN.md](./V1_DATABASE_DDL.zh-CN.md).

The V1 schema includes:

1. `project_binding`
2. `job_binding`
3. `ci_event`
4. `ci_run`
5. `artifact_ref`
6. `failure_case`
7. `decision_record`
8. `healing_attempt`
9. `git_action_log`
10. `audit_log`

Design goals:

1. webhook idempotency and traceability
2. multi-job parallel run tracking
3. locator-failure classification persistence
4. git branch/MR/revert auditing
