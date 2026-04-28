# PHASE-04-PR-IMPACT-ANALYSIS（中文）

## 目标
从前端 diff 预测影响面，并仅执行最小相关回归集。

## 输入
- PR/commit diff
- AST 图谱与 route 绑定
- 场景映射与历史风险信号

## 输出
- 影响分析报告（routes/elements/scenarios）
- 受影响区域风险分
- 回归测试子集选择结果

## 准出标准
- 至少一个 PR 运行采用影响驱动的子集执行。
- 报告质量可支撑 reviewer 决策。

## 非本阶段范围
- 完全替代发布门禁中的全量回归策略。
