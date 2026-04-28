# PHASE-02-RUNTIME-INVENTORY-VERIFICATION（中文）

## 必须检查项
1. `R02-01` 试点路由均完成运行态快照采集。
   证据：`page_snapshot.html`、`accessibility_snapshot.json`、截图文件。
2. `R02-02` 元素画像包含语义 ID 与业务意图。
   证据：元素画像数据集产物。
3. `R02-03` 证据元数据包含 trace 与 commit 关联字段。
   证据：结构化运行记录中的元数据字段。
4. `R02-04` 敏感字段按策略脱敏。
   证据：脱敏日志或 sanitizer 报告。

## 通过规则
四项全部通过。

## 审核角色
运行态工具负责人。
