# AGENTS.md（中文）

## 使命
构建一个可审计、可验证的 AI 辅助 UI 自愈测试平台，面向：
- Java + BDD + Playwright 自动化资产
- React 18 + Ant Design 5 前端变更

## 当前基线（2026-04-28）
以下文档是当前执行口径的最高优先级：
1. `V1_API_CONTRACT.zh-CN.md`
2. `V1_DATABASE_DDL.zh-CN.md`
3. `V1_LANGGRAPH_STATE_MACHINE.zh-CN.md`

执行模式：
1. 平台是独立控制面，与被服务仓库解耦。
2. Jenkins webhook 是主触发入口。
3. V1 默认仅做失败分析，不写补丁。
4. V1.5 通过开关启用分支补丁流。
5. 回退冲突只打标并停止。
6. 统一时区为 `Asia/Shanghai`。

当前仓库以方案与规划为主。此阶段优先沉淀架构、计划、验证规则与治理文档。

## 工作模式
采用分阶段门禁流程：
1. 探索
2. 规划
3. 在隔离范围内实施
4. 用可执行证据验证
5. 评审并进入 L1/L2/L3 分流
6. 回写知识资产

## 提示结构（必需）
每个中大型任务应明确：
- Goal（目标）
- Context（上下文：文件、路由、失败场景、日志）
- Constraints（约束：安全、架构、修改边界）
- Done-when（完成标准：必须通过的测试/证据）

## 强制护栏
- LLM 只负责给出候选；是否可用由运行时验证决定。
- 业务流程变化不能被当作纯 locator 变更自动修复。
- 不允许通过弱化断言来“跑绿”。
- L1 locator 修复不改 Cucumber 业务步骤语义。
- 优先改 Page Object / locator registry，且保持最小 diff。
- L2/L3 风险必须人工审批。

## 验证要求
无证据不算完成。最少证据类型：
- `failure_diagnosis.json`
- `locator_candidates.json`
- `validation_results.json`
- `java_patch.diff`（若产生补丁）
- `rerun_result.json`
- `healing_report.md`

## Skill 使用
相关任务优先使用本地项目 skill：
- `.codex/skills/self-healing-ui/SKILL.md`

新增 skill 时遵循：
- 一个 skill 只做一类工作
- 描述中明确触发语句
- 严格定义输入输出
- 仅在能提升可靠性时才引入脚本/资产

## 项目导航
请结合以下文件：
- `PLAN.md`：阶段路线图
- `PROJECT_STRUCTURE.md`：目录架构
- `TEST_STRATEGY.md`：验证策略
- `phases/`：阶段执行说明
- `verification/`：阶段验收检查

## 完成定义
只有同时满足以下条件才算完成：
- 符合当前阶段边界
- 该阶段验证项通过
- 证据产物可审查
- 风险已记录并分流到正确审批等级
