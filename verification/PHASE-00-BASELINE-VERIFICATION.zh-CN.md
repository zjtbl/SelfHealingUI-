# PHASE-00-BASELINE-VERIFICATION（中文）

## 必须检查项
1. `B00-01` 项目绑定配置存在且字段完整。
   证据：`config/project_binding.example.yaml` 或项目实际绑定文件。
2. `B00-02` 试点范围具备 route-page-scenario 映射。
   证据：`artifacts/<trace_id>/` 下映射产物。
3. `B00-03` 已生成包含文件和行号的 locator 清单。
   证据：`artifacts/<trace_id>/locator_inventory.json`。
4. `B00-04` 至少一个冒烟场景运行成功并产出证据包。
   证据：`rerun_result.json` 与截图/trace 引用。

## 通过规则
四项全部通过。

## 审核角色
QA 架构负责人。
