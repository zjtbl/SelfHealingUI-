# AI 辅助自愈性 UI 自动化测试执行阶段

## 1. 当前目录背景理解

当前目录是一个方案与原型目录，不是 Java 自动化源码目录。已有文件包括：

- `架构.md`：Java + BDD + Playwright UI 自愈测试总体架构。
- `konwledge-rag.md`：UI 元素、测试资产、执行证据、修复历史的知识库与 RAG 方案。
- `mvp-demo.md`：Python/LangGraph MVP 伪代码，覆盖失败分析、元素召回、候选 locator 生成、Playwright 验证、Java 补丁和重跑。
- `UI-Root.html`：可交互架构图，按触发层、决策层、自愈层、执行层、记录层、知识库拆分节点。

你的目标不是单点修复 locator，而是给历史 Java + BDD + Playwright 自动化项目增加 AI 辅助能力，形成“前端变更影响分析、运行失败自愈、Java 补丁生成、重跑验证、知识库回流、新用例生成”的闭环。

## 2. 外部调研结论

调研时间：2026-04-27。

可参考方向：

- Playwright 官方 locator 策略建议优先使用 `getByRole`、`getByLabel`、`getByPlaceholder`、`getByText`、`getByTestId`，并依赖 locator 的自动等待、重试和 strictness 机制。
- Playwright MCP 使用 accessibility snapshot 给 LLM 提供结构化页面上下文，这比单纯截图更适合做可解释的页面元素扫描和候选 locator 生成。
- Healenium 是 Selenium 生态中典型的自愈方案，核心思想是历史元素画像、失败后替代 locator、报告和持久化，但它不能直接覆盖 Java + Playwright + BDD + React/AntD 的企业级链路。
- BrowserStack 等商业平台也把 self-healing 定位为 locator 变化后的自动恢复能力，说明该方向落地价值明确，但你的方案需要更强的本地知识库、审计、代码补丁和人工审批能力。

本项目建议吸收 Healenium 的历史元素记忆思想、Playwright 的真实运行时验证能力、Repo-Map/AST 的代码结构理解能力，以及 RAG 的历史修复案例召回能力。

## 3. 阶段 0：项目基线梳理

目标：把“历史自动化项目”和“前端项目”建立可追溯关系。

输入：

- Java + BDD + Playwright 自动化项目。
- `.feature`、step definition、Page Object、locator 常量或封装。
- 前端 React 18 + Ant Design 5 项目。
- 环境 URL、登录账号、权限、CI 命令、报告路径。

产出：

- `project_binding.yaml`：前端仓库、自动化仓库、base_url、测试命令、默认分支、路由和 feature 映射。
- 初始 `route_map.json`：route、页面 key、源码文件、feature 文件关系。
- 初始 `locator_inventory.json`：Java 代码中所有 locator、文件、行号、调用动作、风险等级。

验收：

- 能从一个 Java locator 反查到 feature、step、Page Object 方法、页面 route。
- 能跑通一个指定场景，并归档日志、截图、trace 或 Allure 报告。

## 4. 阶段 1：Locator 自愈 MVP

目标：先打通“失败 -> 候选 locator -> 验证 -> Java 补丁 -> 重跑 -> 报告”的最小闭环。

推荐流程：

```text
读取 Java BDD 失败日志
  -> 解析失败 step、旧 locator、文件行号、失败类型
  -> 恢复或重新进入失败页面状态
  -> 采集 DOM、accessibility snapshot、截图
  -> 从知识库召回历史元素画像
  -> 生成 3 个以上候选 locator
  -> Playwright 验证唯一性、可见性、可点击/可输入
  -> 生成 Java Page Object 最小 diff
  -> 重跑失败场景
  -> 写入 healing_report.md 和数据库
```

MVP 自动修复范围：

- `LOCATOR_NOT_FOUND`
- `LOCATOR_NOT_UNIQUE`
- `ELEMENT_NOT_VISIBLE`
- `ELEMENT_NOT_ENABLED`
- `ELEMENT_OBSCURED`

MVP 不自动修复范围：

- 业务流程变化。
- 断言语义变化。
- 后端或测试数据问题。
- 权限、网络、环境不稳定。

关键规则：

- LLM 只生成候选，不直接决定修复成功。
- 候选必须由 Playwright 真实浏览器验证。
- Java 补丁只允许修改 locator 或合理等待，不改业务步骤和断言语义。
- 初期只生成 diff 或 PR，不自动合并主干。

## 5. 阶段 2：页面元素扫描与知识库入库

目标：把运行时页面元素沉淀成可查询、可回放、可向量检索的资产。

采集内容：

- URL、title、route、页面状态。
- DOM 片段、accessibility snapshot。
- 可交互元素：button、link、input、textarea、select、tab、menu、dialog、table、form item。
- 元素属性：tag、role、accessible name、label、placeholder、text、href、name、type、`data-testid`。
- 元素位置：bounding box、可见性、启用状态、可编辑状态。
- 证据：全页截图、元素截图、trace、console error、network failure。

结构化库存储：

- PostgreSQL 或 SQLite 保存 page、element、locator_version、test_asset、step_element_binding、execution_run、healing_event。
- 对象存储保存 screenshot、trace、DOM、a11y snapshot。
- 向量库存储元素语义文档、BDD step 文档、失败案例摘要、前端组件摘要。

推荐元素主键：

```text
element_uid = 业务域 + 页面 + 容器 + 元素意图 + 元素类型
```

不要把 CSS 或 XPath 当主键。locator 是版本化属性，元素语义 ID 才是长期资产。

## 6. 阶段 3：前端 AST 与 Repo-Map 能力

目标：让系统理解 React/AntD 源码结构，支持 PR 影响分析。

扫描内容：

- route 配置。
- 页面组件、容器组件、公共组件。
- AntD 组件：Button、Input、Select、Table、Modal、Form.Item、DatePicker、Dropdown。
- JSX 属性：`data-testid`、`aria-label`、`role`、`name`、`placeholder`、`label`、按钮文本。
- 事件处理：onClick、onChange、onFinish、onSubmit。
- 组件到 route、元素、测试场景的映射关系。

产出：

- `component_graph.static.json`
- `route_map.json`
- `locator_catalog.json`
- `test_impact_report.json`

作用：

- PR 提交后判断哪些页面、元素、BDD 场景可能受影响。
- 识别缺少 `data-testid` 或依赖脆弱 CSS/XPath 的元素。
- 在自愈失败时辅助判断是 locator 变化、组件变化，还是业务流程变化。

## 7. 阶段 4：PR 级影响分析

目标：从“失败后才修”升级为“提交后先预测影响并选择最小回归集合”。

流程：

```text
前端 PR / commit diff
  -> AST 识别变更组件和元素
  -> 图谱查询关联 route、element、scenario
  -> 计算 locator 风险
  -> 选择最小回归测试集
  -> 执行受影响 Java BDD 场景
  -> 成功/失败结果回流知识库
```

风险评分因素：

- `data-testid` 是否删除或重命名。
- label、placeholder、button text 是否变化。
- AntD 组件类型是否变化。
- DOM 层级和 portal scope 是否变化。
- 历史 locator 是否依赖 CSS、XPath、nth、动态 class。
- 该元素是否属于核心业务流。

## 8. 阶段 5：Java 补丁与治理闭环

目标：让 AI 生成的修改可审计、可重跑、可回滚。

补丁策略：

- 优先修改 Page Object 或 locator registry。
- 避免直接修改 Cucumber step 业务流程。
- 使用 JavaParser 或 tree-sitter-java 做 AST 级定位，MVP 可先做 Page Object 方法级安全替换。
- 补丁必须附带验证证据：候选 locator、验证结果、重跑结果、截图/trace 引用。

分级策略：

- L1：locator、testid、合理等待策略，可自动生成 PR。
- L2：断言语义、业务预期、流程变化，必须人工审批。
- L3：环境、后端、数据、权限问题，阻断并输出诊断证据。

## 9. 阶段 6：新故事卡自动生成测试

目标：从自愈扩展到“新增需求自动生成 BDD 和 Java Playwright 测试草稿”。

流程：

```text
故事卡 / 验收标准
  -> 关联前端 diff 和新增 route
  -> Playwright 探索运行态页面
  -> 建立新元素画像
  -> 向量召回相似历史 BDD 场景
  -> 生成 Gherkin 场景
  -> 复用已有 step definition
  -> 必要时生成 Java Page Object / step 草稿
  -> dry-run + 浏览器验证
  -> 人工 review 入库
```

原则：

- 先生成 BDD 场景，再生成 Java 代码。
- 优先复用已有 step。
- 不编造运行态页面不存在的元素。
- 初期必须人工 review。

## 10. 阶段 7：企业级治理与度量

目标：支持多项目推广、审计、脱敏和 ROI 度量。

能力：

- Prompt、输入 hash、输出 hash、模型版本、审批记录全量审计。
- DOM、截图、trace 入库前脱敏。
- 对象存储权限隔离和 TTL。
- 自愈成功率、重跑通过率、人工接受率、误修复率、平均修复耗时、脆弱 locator 数量趋势。
- 自动 PR 评论：影响页面、影响用例、locator 风险、建议补充 `data-testid`。

## 11. 推荐近期落地顺序

1. 在历史 Java 自动化项目上实现 locator 扫描器，导出 `locator_inventory.json`。
2. 选 1 个 AntD 页面和 10 到 50 个 BDD 场景，手工建立 `project_binding.yaml`。
3. 用 Playwright 采集该页面的 DOM、a11y snapshot 和截图，建立元素画像。
4. 构建失败日志解析器，只支持 3 到 5 类 locator 失败。
5. 让 LLM 输出结构化候选 locator JSON。
6. 用 Playwright 验证候选，禁止未验证候选进入补丁。
7. 生成 Java Page Object 最小 diff。
8. 重跑单场景并生成 `healing_report.md`。
9. 将成功和失败样本都写入知识库。
10. 再扩展前端 AST 扫描和 PR 影响分析。

## 12. 本地 skill

已创建本地 Codex skill：

```text
/mnt/f/codex-work/SelfHealingUI/.codex/skills/self-healing-ui
```

用途：

- 分析 Java + BDD + Playwright 历史自动化项目。
- 设计 locator 自愈 MVP。
- 扫描页面元素并规划结构化库/向量库入库。
- 生成自愈报告、补丁治理、PR 影响分析步骤。

Windows 路径：

```text
F:\codex-work\SelfHealingUI\.codex\skills\self-healing-ui
```

重启 Codex 后可重新发现项目级 skill。
