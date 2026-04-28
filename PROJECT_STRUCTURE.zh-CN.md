# PROJECT_STRUCTURE.md（中文）

## 顶层结构
```text
SelfHealingUI/
  AGENTS.md
  AGENT.md
  SKILLS.md
  PLAN.md
  PROJECT_STRUCTURE.md
  TEST_STRATEGY.md
  docs/
    architecture/
    workflows/
  phases/
  verification/
    templates/
  config/
  platform/
    orchestrator/
    agents/
    scanners/
    healing/
    runners/
    storage/
    models/
    prompts/
  tests/
    contract/
    integration/
    e2e/
  artifacts/
```

## 目录职责
- `docs/architecture/`：架构决策、数据边界、信任边界。
- `docs/workflows/`：从触发到证据的流程定义。
- `phases/`：各阶段执行说明。
- `verification/`：各阶段验收检查。
- `config/`：机器可读策略与项目绑定配置。
- `platform/`：后续实现模块骨架。
- `tests/`：验证代码分层（后续实现）。
- `artifacts/`：运行证据归档。

## 命名规范
- 英文文件：`NAME.md`
- 中文配套：`NAME.zh-CN.md`

## 归属建议
- 策略类文档：测试架构负责人
- 配置模板：平台负责人
- 验收文件：QA 治理负责人
