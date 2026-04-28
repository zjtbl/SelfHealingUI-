---
name: self-healing-ui
description: 面向 Java + BDD + Playwright 项目的 AI 辅助 UI 自愈设计与实施技能。用于分析历史自动化资产、扫描 locator 与页面元素、设计 locator 自愈流程、构建知识库/向量检索、生成 Java locator 补丁、规划 React/Ant Design 影响分析及自愈落地步骤。
---

# Self-Healing UI（中文）

## 工作模型
把自愈当作“可验证的工程化 harness”，而不是直接让 LLM 改代码。

核心闭环：

```text
理解仓库状态
  -> 选择工作流模式
  -> 扫描资产
  -> 分类风险或失败
  -> 采集运行证据
  -> 召回历史上下文
  -> 生成 locator/补丁候选
  -> 用 Playwright 验证
  -> 仅在安全区域打补丁
  -> 重跑受影响测试
  -> 回写证据与决策
```

## 核心分离原则
- **Intent**：业务动作或断言意图
- **Element**：稳定的 `element_uid` 与语义画像
- **Locator**：可版本化的实现细节
- **Evidence**：DOM、a11y、截图、trace、日志
- **Patch**：最小 Java/前端 diff

## 当前交付口径（2026-04-28）

当前执行基线如下：

1. 平台是独立控制面，与被服务自动化仓库解耦。
2. 主要触发入口是 Jenkins webhook 回调平台 API。
3. 平台也支持主动触发 Jenkins Job。
4. GitLab 自动化范围：建分支、提交补丁、创建 Draft MR、查询 MR 变更、revert commit。
5. 不自动合并主干分支。
6. V1 仅处理 locator 类失败；断言语义变更不在当前范围。
7. 回退冲突时只打标失败并停止自动流程。
8. 时区统一为 `Asia/Shanghai`。
9. 一个项目可配置多个 Jenkins Job，并行运行必须以 `event_id`/`run_id` 可追溯。

## 会话启动协议
1. 确认 `pwd` 在项目内。
2. 读取 `steps.md`、`skill-self-think.md` 与相关配置。
3. 检查最近进度与产物（如 `artifacts/`、执行计划目录）。
4. 明确一个 workflow mode 和一个主输出产物。
5. 先声明完成所需的验证命令或证据，再开始执行。

## 关键规则
- 仅生成文档或补丁不代表完成，必须有验证证据。
- 规则与 Playwright 决定候选是否有效。
- LLM 负责候选生成、意图解释、补丁草案。
- 所有产物必须绑定 `trace_id`、`commit_sha`、`source_repo`、`tool_version`。
- 使用完成状态：`DONE`、`DONE_WITH_CONCERNS`、`BLOCKED`、`NEEDS_CONTEXT`。
