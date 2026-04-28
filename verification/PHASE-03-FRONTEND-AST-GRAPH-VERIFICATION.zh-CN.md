# PHASE-03-FRONTEND-AST-GRAPH-VERIFICATION（中文）

## 必须检查项
1. `A03-01` AST 扫描可将变更文件映射到组件。
   证据：组件图谱快照产物。
2. `A03-02` 变更组件具备对应 route 绑定。
   证据：route-component 映射产物。
3. `A03-03` 输出含 locator 脆弱性风险信号。
   证据：风险字段报告。
4. `A03-04` 可识别契约缺失（`data-testid`/role/label）。
   证据：静态分析告警清单。

## 通过规则
四项全部通过。

## 审核角色
前端可测性负责人。
