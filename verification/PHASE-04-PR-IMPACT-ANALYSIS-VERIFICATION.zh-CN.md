# PHASE-04-PR-IMPACT-ANALYSIS-VERIFICATION（中文）

## 必须检查项
1. `I04-01` PR diff 可成功接入并生成变更文件清单。
   证据：diff 摘要产物。
2. `I04-02` 影响报告包含 routes、elements、scenarios 与风险分。
   证据：`test_impact_report.json`。
3. `I04-03` 最小回归子集已生成并执行。
   证据：子集列表与执行结果。
4. `I04-04` locator 失败可自动路由到自愈流程。
   证据：trace 关联与失败路由记录。

## 通过规则
四项全部通过。

## 审核角色
CI 质量负责人。
