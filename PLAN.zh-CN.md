# PLAN.md（中文）

## 目标
基于现有项目方案文档与 agent 最佳实践，定义可落地的 UI 自愈测试平台实施路线。

## 阶段路线图
1. Phase 00：基线绑定
2. Phase 01：Locator 自愈 MVP
3. Phase 02：运行态元素资产化
4. Phase 03：前端 AST 与仓库图谱
5. Phase 04：PR 影响分析
6. Phase 05：Java 补丁治理
7. Phase 06：故事卡到测试生成
8. Phase 07：企业级治理与度量

## 交付规则
- 每个阶段必须包含：
  - `phases/` 下的阶段说明
  - `verification/` 下的阶段验证文件
  - 明确准入/准出条件
  - 必需证据产物

## 可追溯规则
- 每次运行必须绑定：
  - `trace_id`
  - `commit_sha`
  - `source_repo`
  - `tool_version`
  - `created_at`

## 风险分流
- L1：locator / 测试契约 / 等待策略
- L2：断言语义或业务流程语义
- L3：环境、数据、后端、权限

## 近期里程碑
1. 完成 Phase 00 的产物与验证证据。
2. 在隔离范围内完成 Phase 01（1 个页面、10-50 个场景）。
3. 输出首批可审计自愈报告证据集。
