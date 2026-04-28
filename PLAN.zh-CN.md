# PLAN.md（中文）

## 目标
基于现有项目方案文档与 agent 最佳实践，定义可落地的 UI 自愈测试平台实施路线。

## 执行基线
1. V1 范围：独立控制面 + Jenkins webhook 接入 + 失败分析 + 决策记录。
2. V1.5 范围：通过特性开关启用 locator 分支自愈流。
3. GitLab 合并主干始终保持人工执行。

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
1. 完成单项目 webhook 接入、上下文补全与工件拉取链路。
2. 在“仅分析模式”下完成 locator 失败分类与决策落库。
3. 在受控试点开启 V1.5 分支流并验证回退行为。
