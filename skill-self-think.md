# UI 自愈测试项目 Skill 增强思考：引入 Harness Engineering 与 gstack 实践

## 1. 背景

当前目录里的项目目标已经比较清晰：给历史 Java + BDD + Playwright 自动化项目增加 AI 辅助能力，围绕 React 18 + Ant Design 5 前端变化，完成 UI locator 自愈、页面元素扫描、知识库/RAG、Java 补丁生成、重跑验证、PR 影响分析和新故事卡自动化生成。

之前的项目级 skill 已经覆盖了“自愈测试领域规则”，例如：

- locator 策略。
- Java Page Object 补丁规则。
- Playwright 运行时验证。
- UI 元素知识库结构。
- React/AntD AST 影响分析。

这次增强的重点不是替换这些规则，而是引入 harness engineering 的工程化组织方式，让 AI agent 在项目里更可靠地工作。

核心变化：

```text
从：一个懂 UI 自愈的 skill
到：一个能组织规划、扫描、验证、修复、审查、沉淀的项目级工程 harness
```

## 2. 外部资料调研摘要

调研时间：2026-04-28。

### 2.1 gstack 的工程实践启发

gstack 的关键价值不在于某个单独提示词，而在于它把 AI 编码组织成“虚拟工程团队”。它包含 CEO、工程经理、设计、review、QA、安全、发布等多个角色，并通过 slash commands 触发不同角色。

对本项目有价值的实践：

- 角色路由：先判断任务是产品澄清、工程评审、测试验证、代码审查、发布，还是复盘。
- 阶段流水线：Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect。
- 上一阶段产物喂给下一阶段：设计文档、测试计划、review 结果、QA 证据、发布报告不丢失。
- QA 必须使用真实浏览器，而不是只读代码或只相信模型。
- 工程实践要有 completion status，例如完成、带风险完成、阻塞、缺上下文。
- 项目记忆、操作经验、遥测和复盘要沉淀为可复用资产。
- 不把 agent 当“会写代码的聊天框”，而是当一个需要流程、工具、证据和门禁约束的工程成员。

参考：

- gstack GitHub: https://github.com/garrytan/gstack
- gstack README 中列出的角色和 sprint 流程：https://github.com/garrytan/gstack
- gstack browse skill: https://github.com/garrytan/gstack/blob/main/SKILL.md
- gstack project instructions: https://github.com/garrytan/gstack/blob/main/CLAUDE.md

### 2.2 OpenAI Harness Engineering 的启发

OpenAI 的 harness engineering 文章强调：模型能力不是唯一变量，agent 运行的环境、工具、上下文、验证机制和反馈回路同样决定结果质量。

对本项目有价值的实践：

- 仓库知识库应该成为 system of record。
- 顶层说明文件只做地图，不做 1000 页百科。
- 文档、计划、质量分、可靠性、安全规则都应该版本化并放在仓库里。
- UI、日志、metrics、trace 要变成 agent 可读的环境。
- 架构边界和工程 taste 不能只写在文档里，要用 lint、结构测试、CI 规则固化。
- 人类的工作重心从“手写代码”转为“设计环境、指定意图、建立反馈回路”。

参考：

- OpenAI: Harness engineering: leveraging Codex in an agent-first world  
  https://openai.com/index/harness-engineering/

### 2.3 Anthropic 长任务 Harness 的启发

Anthropic 的 long-running agent 文章强调：长任务失败往往不是模型不会写，而是没有上下文交接、没有 feature list、没有进度文件、没有每轮验证。

对本项目有价值的实践：

- 初始化 agent 应该创建 `init.sh`、进度文件、feature list。
- 后续 agent 每次只做一个可验证切片。
- 每次会话开始先读 `pwd`、git log、进度文件、feature list。
- 每次结束要写进度，留下干净状态。
- feature 的完成状态建议用 JSON，而不是随意 Markdown。
- 浏览器端到端验证是防止“假完成”的关键。

参考：

- Anthropic: Effective harnesses for long-running agents  
  https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic: Harness design for long-running application development  
  https://www.anthropic.com/engineering/harness-design-long-running-apps

## 3. 对当前项目的 Skill 增强方向

### 3.1 现有 skill 的定位

当前项目级 skill：

```text
.codex/skills/self-healing-ui/SKILL.md
```

它适合作为领域执行规则，重点回答：

- 如何扫描 Java BDD Playwright locator。
- 如何判断 locator 风险。
- 如何生成候选 locator。
- 如何用 Playwright 验证候选。
- 如何生成 Java 最小补丁。
- 如何构建 UI 元素知识库。

### 3.2 增强后的 skill 定位

增强后，它应该同时回答：

- 当前任务属于哪个工程阶段。
- 应该由哪类“角色”视角处理。
- 输入和输出工件是什么。
- 哪些证据能证明完成。
- 哪些情况必须停止或升级给人。
- 哪些经验应该写回项目记忆。
- 如何避免 agent 在长任务中丢上下文、假完成、误修复。

## 4. 推荐的项目级 Harness 文件结构

建议逐步把项目从“几个 Markdown 方案文件”升级成“agent 可读、可验证、可演进的工程仓库”。

```text
SelfHealingUI/
  AGENTS.md
  steps.md
  skill-self-think.md
  self-healing-progress.md
  self_healing_features.json

  .codex/
    skills/
      self-healing-ui/
        SKILL.md
        agents/openai.yaml
        references/research-notes.md

  config/
    project_binding.yaml
    locator_policy.yaml
    repair_policy.yaml
    env_policy.yaml

  docs/
    harness/
      CORE_BELIEFS.md
      WORKFLOWS.md
      QUALITY_SCORE.md
      RELIABILITY.md
      SECURITY.md
    exec-plans/
      active/
      completed/
    runbooks/
      java-bdd-runner.md
      playwright-runtime-scan.md
      locator-healing.md

  artifacts/
    {trace_id}/
      locator_inventory.json
      page_snapshot.html
      accessibility_snapshot.json
      locator_candidates.json
      validation_results.json
      java_patch.diff
      rerun_result.json
      healing_report.md

  scripts/
    init.sh
    verify.sh
    scan-locators.sh
    scan-runtime.sh
    validate-skill.sh
```

说明：

- `AGENTS.md` 只做地图，告诉 agent 该读哪些文件，不塞所有细节。
- `self_healing_features.json` 管理项目阶段和验收项，避免“看起来完成”。
- `self-healing-progress.md` 记录阶段性进度、决策和阻塞。
- `docs/harness/*` 存放长期工程原则。
- `config/*` 存放可执行策略。
- `artifacts/{trace_id}` 存放每次运行证据。
- `scripts/*` 把常用验证动作变成可重复命令。

## 5. 建议的 Workflow 路由

借鉴 gstack 的角色路由，但不要机械复制它的 slash command。当前项目可以使用以下工作模式。

| 模式 | 作用 | 典型输出 |
| --- | --- | --- |
| Product Challenge | 澄清自愈平台要解决的真实痛点、验收指标和边界 | `docs/exec-plans/active/*.md` |
| Engineering Plan Review | 评审架构、数据流、测试性、扩展性和风险 | 方案评审意见、风险清单 |
| Baseline Import | 建立前端仓库、Java 自动化仓库、route、feature、CI 的初始映射 | `project_binding.yaml` |
| Locator Scan | 扫描 Java Playwright locator，标记脆弱度 | `locator_inventory.json` |
| Runtime Scan | 用 Playwright 扫页面 DOM/a11y/screenshot/trace | `page_snapshot.html`、`accessibility_snapshot.json` |
| Healing Loop | 失败分类、候选生成、浏览器验证、Java 补丁、重跑 | `healing_report.md`、`java_patch.diff` |
| Impact Analysis | 从前端 diff 找受影响 route、element、scenario | `test_impact_report.json` |
| Review | 查误修复、弱断言、缺证据、数据泄露、缺测试 | review report |
| QA | 像真实用户一样跑关键路径，附截图和 trace | QA evidence |
| Ship | 检查门禁，准备 PR、发布说明和回滚方案 | PR 描述、release note |
| Retro | 把失败和人工修正转成规则、脚本或 skill 更新 | `self-healing-progress.md` 更新 |

## 6. 增强后的 Skill 核心规则

### 6.1 会话启动规则

每次处理项目任务时，先做这个最小启动流程：

```text
1. 确认 pwd。
2. 读取 steps.md。
3. 读取 skill-self-think.md。
4. 读取 .codex/skills/self-healing-ui/SKILL.md。
5. 如果存在，读取 self-healing-progress.md 和 self_healing_features.json。
6. 明确当前 workflow mode。
7. 明确本轮只交付一个主要 artifact。
8. 明确完成所需验证证据。
```

### 6.2 完成状态规则

每次输出结论时使用明确状态：

```text
DONE：已完成并有证据。
DONE_WITH_CONCERNS：已完成，但存在已说明的风险或缺口。
BLOCKED：无法继续，说明尝试过什么、卡在哪里。
NEEDS_CONTEXT：缺少必要输入，说明需要用户提供什么。
```

### 6.3 自愈安全规则

AI 不直接决定修复成功。

必须满足：

- 候选 locator 由 Playwright 验证唯一、可见、可交互。
- Java 补丁是最小 diff。
- 重跑失败场景通过。
- 报告包含旧 locator、新 locator、原因、证据、风险、回滚方式。

禁止：

- 为了通过测试弱化断言。
- 把业务变化当 locator 变化自动修。
- 默认加 `force:true`。
- 默认加长 sleep。
- 未经验证直接写入主知识库。

### 6.4 知识库规则

知识库不是“把所有东西塞进向量库”。

应该分层：

- 结构化库：事实、关系、版本、执行记录。
- 向量库：语义召回、相似案例、历史步骤、失败经验。
- 对象存储：截图、trace、DOM、a11y、视频、diff。
- 仓库文档：工程规则、计划、进度、风险和复盘。

每个 artifact 至少绑定：

```text
trace_id
commit_sha
source_repo
tool_version
created_at
```

### 6.5 反馈回流规则

每次人工 review、失败自愈、误修复、环境故障都要问：

```text
这次问题是一次性问题，还是应该沉淀成规则？
如果沉淀，应该进入哪里？
- locator_policy.yaml
- repair_policy.yaml
- env_policy.yaml
- SKILL.md
- docs/harness/*
- scripts/*
- self_healing_features.json
```

## 7. 相比之前 skill 的优势

### 7.1 从领域规则升级到工程系统

之前 skill 更像“UI 自愈测试专家手册”，现在增强为“项目级 harness”。它不仅告诉 agent 怎么修 locator，还告诉 agent 怎么开始、怎么判断阶段、怎么验证、怎么留下证据、怎么结束。

### 7.2 降低长任务上下文丢失

新增会话启动协议、进度文件、feature JSON、exec plan 目录后，agent 不需要依赖上一轮聊天上下文。即使重启会话，也能从仓库恢复任务状态。

### 7.3 防止假完成

之前容易出现“生成了方案或补丁就算完成”。增强后要求每个阶段有验收证据，例如：

- locator scan 必须有 `locator_inventory.json`。
- runtime scan 必须有 DOM/a11y/screenshot。
- healing loop 必须有 validation、patch、rerun。
- ship 必须有测试结果和回滚方案。

### 7.4 强化浏览器验证

gstack 和 Anthropic 的实践都说明，浏览器端到端验证能发现代码阅读发现不了的问题。对 UI 自愈来说，这一点更重要，因为 locator 是否真的可用必须在页面状态里验证。

### 7.5 让知识库更可维护

之前的知识库方案偏数据模型。增强后补充了“仓库知识是 system of record”的理念：顶层文档做地图，细节分层存放，并用 lint、脚本和 CI 检查漂移。

### 7.6 让人工经验可复利

每次人工纠正不只是修一次 bug，而是沉淀为 policy、script、skill 或 docs。这样项目越做越容易被 agent 理解和验证。

### 7.7 更适合企业落地

增强后补充了完成状态、升级规则、审计证据、敏感数据 stop-gate、人工审批边界。这比单纯 locator 自愈更适合金融、政企、内部系统自动化项目。

## 8. 项目实践中应该注意什么

### 8.1 不要直接照搬 gstack

gstack 是 Claude Code 的通用工程团队 skill 包。你的项目是 Java + BDD + Playwright + React/AntD 的垂直场景。应该借鉴它的角色和流水线思想，而不是复制所有命令。

建议：

- 保留自愈测试领域规则。
- 引入角色路由和阶段门禁。
- 用项目自己的 artifact 和 CI 命令替代 gstack 的默认命令。

### 8.2 不要把 skill 写成超长百科

OpenAI 的 harness 经验里，一个巨大说明文件会挤占上下文、快速腐烂、难以验证。

建议：

- `SKILL.md` 放核心流程和门禁。
- `docs/harness/*` 放长期规范。
- `config/*.yaml` 放机器可读策略。
- `artifacts/*` 放运行证据。

### 8.3 不要先追求全自动

MVP 阶段不要做自动合并、全量系统扫描、全自动故事卡生成。

先完成：

```text
一个页面
10-50 个历史 BDD 场景
3-5 类 locator 失败
Playwright 验证
Java 最小补丁
人工 review
```

### 8.4 不要让向量库承担事实存储职责

向量库适合召回相似元素、相似步骤、相似失败，不适合作为 locator 当前版本、执行结果、审批记录的权威数据源。

建议：

- PostgreSQL/SQLite 存事实。
- 图关系存影响链路。
- 向量库存语义文档。
- 对象存储保存证据。

### 8.5 不要把业务变化误修成 locator 自愈

如果前端真的新增字段、改变流程、改变断言语义，自动修改 locator 是危险的。

建议：

- L1：locator/testid/等待，允许自动 PR。
- L2：业务预期/断言/流程，必须人工审批。
- L3：环境/数据/后端，阻断并报告。

### 8.6 不要忽略敏感数据

DOM、截图、trace、视频可能包含客户名、账号、交易金额、内部 URL、token。

建议：

- 入库前脱敏。
- artifact 设置 TTL。
- prompt 记录 hash，不默认保留原文敏感内容。
- 高风险页面禁止直接进入长期向量库。

### 8.7 不要只度量“生成了多少代码”

UI 自愈项目应该度量有效结果。

建议指标：

- locator 类失败自愈成功率。
- 自愈后重跑通过率。
- 自动 PR 人工接受率。
- 误修复率。
- 平均修复耗时下降。
- 脆弱 locator 数量下降。
- PR 影响分析命中率。
- 新故事卡生成后人工修改比例。

## 9. 推荐下一步

### 9.1 更新项目文档

建议新增：

```text
AGENTS.md
self-healing-progress.md
self_healing_features.json
docs/harness/CORE_BELIEFS.md
docs/harness/WORKFLOWS.md
docs/harness/QUALITY_SCORE.md
docs/harness/RELIABILITY.md
docs/harness/SECURITY.md
```

### 9.2 建立 MVP feature list

建议先写 `self_healing_features.json`：

```json
[
  {
    "id": "baseline.project-binding",
    "category": "baseline",
    "description": "Bind one frontend repo, one Java BDD automation repo, one base URL, and one test command",
    "evidence": ["config/project_binding.yaml"],
    "passes": false
  },
  {
    "id": "scan.java-locators",
    "category": "locator-scan",
    "description": "Scan Java Playwright locators from Page Objects and step definitions",
    "evidence": ["artifacts/{trace_id}/locator_inventory.json"],
    "passes": false
  },
  {
    "id": "runtime.page-snapshot",
    "category": "runtime-scan",
    "description": "Collect DOM, accessibility snapshot, screenshot, and key element profiles for one target page",
    "evidence": ["artifacts/{trace_id}/page_snapshot.html", "artifacts/{trace_id}/accessibility_snapshot.json"],
    "passes": false
  },
  {
    "id": "heal.locator-failure",
    "category": "healing-loop",
    "description": "Heal one locator-level failure, validate candidate with Playwright, patch Java, rerun scenario",
    "evidence": ["artifacts/{trace_id}/validation_results.json", "artifacts/{trace_id}/java_patch.diff", "artifacts/{trace_id}/rerun_result.json"],
    "passes": false
  }
]
```

### 9.3 把 skill 分成核心与引用

当前 `SKILL.md` 已经加入 harness 规则。后续如果继续变长，建议拆分：

```text
.codex/skills/self-healing-ui/
  SKILL.md
  references/
    research-notes.md
    harness-workflows.md
    locator-policy.md
    java-patch-policy.md
    knowledge-schema.md
```

### 9.4 优先实现验证脚本

比继续写方案更重要的是把关键验证动作脚本化：

```text
scripts/init.sh
scripts/verify.sh
scripts/scan-locators.sh
scripts/scan-runtime.sh
```

一旦这些脚本存在，agent 的可靠性会明显提升，因为每次都能用同样方式恢复环境、执行扫描、验证结果。

## 10. 本次已同步更新

本次已增强项目级 skill：

```text
.codex/skills/self-healing-ui/SKILL.md
```

新增内容包括：

- 会话启动协议。
- workflow mode 路由。
- harness rules。
- 推荐项目 harness 文件结构。
- harness 改进输出项。
- completion status 协议。

本文档本身作为更详细的思考与实践指南：

```text
skill-self-think.md
```
