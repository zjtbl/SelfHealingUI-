# PHASE-01-LOCATOR-HEALING-MVP-VERIFICATION（中文）

## 必须检查项
1. `H01-01` 失败解析器可提取 scenario、step、locator、错误类型。
   证据：`artifacts/<trace_id>/failure_diagnosis.json`。
2. `H01-02` 结构化候选 locator 至少 3 个。
   证据：`artifacts/<trace_id>/locator_candidates.json`。
3. `H01-03` 候选验证包含唯一性与可操作性检查。
   证据：`artifacts/<trace_id>/validation_results.json`。
4. `H01-04` 补丁最小且仅修改允许范围。
   证据：`artifacts/<trace_id>/java_patch.diff`。
5. `H01-05` 目标失败场景重跑通过。
   证据：`artifacts/<trace_id>/rerun_result.json`。
6. `H01-06` 自愈报告已生成且可审查。
   证据：`artifacts/<trace_id>/healing_report.md`。

## 通过规则
六项全部通过。

## 审核角色
自动化负责人 + QA 负责人。
