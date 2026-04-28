# UI 自愈测试与 AI 用例生成知识库方案

## 1. 建设目标

知识库不是只存一批 selector，而是建立“业务意图、页面元素、前端实现、自动化步骤、执行证据、修复历史”之间的可追溯关系。它需要同时服务两个目标：

1. UI 元素自愈：当前端代码提交后，能识别历史自动化脚本中可能受影响的定位器；测试失败时，能基于页面现状和历史语义生成候选 locator，经过 Playwright 实际验证后自动修复或生成修复 PR。
2. 新故事卡 UI 自动化生成：当新增页面或功能进入后，能结合需求、设计图、前端代码、运行态页面和历史测试模式，生成 BDD 场景、Java step、Page Object 或 Screenplay task，并由人类 review 微调。

建议把知识库定位为“测试资产智能中台”，而不是单纯的向量库。向量检索只解决语义召回，真正可控的自愈需要结构化索引、图关系、运行时证据和代码补丁闭环。

## 2. 总体数据分层

### 2.1 结构化库

推荐使用 PostgreSQL 作为主数据存储，保存可审计、可版本化、可查询的核心实体。

核心表：

| 表 | 作用 |
| --- | --- |
| `ui_page` | 页面、路由、菜单、所属系统、业务域、权限信息 |
| `ui_component` | React 组件、Ant Design 组件、源码路径、props、父子关系 |
| `ui_element` | 页面元素的稳定语义 ID、角色、文本、标签、placeholder、业务含义 |
| `locator_version` | locator 历史版本，包括 CSS、XPath、role、text、testId、生成原因 |
| `test_asset` | BDD feature、scenario、step definition、Page Object、Java 方法 |
| `step_element_binding` | 自动化步骤和 UI 元素的绑定关系 |
| `commit_change` | 前端提交、变更文件、diff 摘要、影响组件、影响路由 |
| `execution_run` | CI 执行批次、环境、浏览器、分支、构建号 |
| `execution_observation` | 每个失败或关键成功步骤的截图、DOM、a11y tree、trace、stack |
| `healing_event` | 自愈事件、候选 locator、验证结果、置信度、是否被采纳 |
| `review_decision` | 人工审批、驳回原因、合并记录、回滚记录 |

### 2.2 图关系库

推荐使用 Neo4j 或 PostgreSQL + Apache AGE。图关系用于做影响分析和可解释链路。

核心关系：

```text
StoryCard -> Requirement -> Page -> Component -> Element
Element -> LocatorVersion
Scenario -> Step -> StepDefinition -> PageObjectMethod -> LocatorVersion
Commit -> ChangedFile -> Component -> Page -> Element -> Scenario
ExecutionFailure -> Step -> Element -> HealingEvent -> CodePatch
```

典型查询：

```cypher
MATCH (c:Commit {sha: $sha})-[:CHANGED]->(:File)-[:DECLARES]->(comp:Component)
MATCH (comp)-[:RENDERS*0..3]->(e:Element)<-[:TARGETS]-(s:Step)<-[:HAS_STEP]-(sc:Scenario)
RETURN sc, s, e
```

这个查询可以回答：本次前端提交可能影响哪些历史 BDD 场景。

### 2.3 向量库

推荐使用 OpenSearch Vector、Qdrant、Milvus 或企业内部已有向量检索能力。

建议拆成多个 collection，而不是所有内容混在一个 collection。

| Collection | 嵌入内容 | 用途 |
| --- | --- | --- |
| `element_semantic` | 元素语义描述、role、accessible name、label、业务词汇 | 找“等价元素” |
| `frontend_code` | React 组件摘要、props、render 结构、AntD 组件用途 | 变更影响分析 |
| `bdd_steps` | Gherkin 步骤、step definition 摘要、操作意图 | 用例生成和步骤复用 |
| `execution_failure` | 错误堆栈、失败截图描述、DOM 摘要、修复结果 | 失败案例复用 |
| `story_requirement` | 故事卡、验收标准、设计描述 | 新用例生成 |

### 2.4 对象存储

截图、trace、DOM 快照、accessibility snapshot、视频、失败报告不要直接放数据库，建议进入对象存储。

对象类型：

- `screenshot/full_page/{run_id}/{step_id}.png`
- `screenshot/element_crop/{element_id}/{version}.png`
- `trace/{run_id}.zip`
- `dom/{run_id}/{page_id}.html`
- `a11y/{run_id}/{page_id}.json`
- `patch/{healing_id}.diff`

数据库只保存 URI、hash、采集时间和关联实体。

## 3. UI 元素标准模型

每个 UI 元素需要有一个稳定的 `element_uid`。不要直接把 CSS 或 XPath 当主键。

示例：

```json
{
  "element_uid": "trade.search.form.counterparty.input",
  "system": "payments-portal",
  "page_id": "trade-search",
  "route": "/trade/search",
  "business_name": "交易对手方输入框",
  "intent": "输入交易对手方名称或编号以过滤交易列表",
  "role": "textbox",
  "accessible_name": "Counterparty",
  "label": "Counterparty",
  "placeholder": "Search counterparty",
  "visible_text_nearby": ["Counterparty", "Search", "Reset"],
  "component_path": "src/pages/trade/SearchPage.tsx",
  "react_component": "CounterpartySearchInput",
  "antd_component": "Input.Search",
  "dom_signature": {
    "tag": "input",
    "name": "counterparty",
    "type": "text",
    "data-testid": "counterparty-search-input"
  },
  "visual_signature": {
    "bbox": [312, 184, 420, 32],
    "screenshot_uri": "s3://ui-kb/element_crop/..."
  },
  "locator_candidates": [
    {
      "type": "testId",
      "value": "page.getByTestId('counterparty-search-input')",
      "stability_score": 0.95
    },
    {
      "type": "role",
      "value": "page.getByRole('textbox', { name: 'Counterparty' })",
      "stability_score": 0.90
    },
    {
      "type": "css",
      "value": "input[name='counterparty']",
      "stability_score": 0.72
    }
  ],
  "last_seen_commit": "abc123",
  "status": "active"
}
```

定位器优先级建议：

1. 显式测试契约：`data-testid`、业务稳定 ID。
2. 用户可感知语义：`getByRole` + accessible name、`getByLabel`、`getByPlaceholder`。
3. 结构约束：父容器 + role、列表项 + 文本过滤。
4. 视觉或邻近关系：附近 label、按钮文本、布局位置。
5. CSS/XPath：只作为兜底，且需要记录脆弱原因。

## 4. 知识库构建流程

### 4.1 前端静态扫描

输入：

- React 18 代码。
- Ant Design 5 组件。
- 路由配置。
- 提交 diff。
- 部分需求描述和故事卡。

处理：

1. 使用 AST 扫描 JSX/TSX，不建议纯正则。
2. 识别路由、页面组件、容器组件、表单组件、按钮、表格列、弹窗、菜单。
3. 提取 `data-testid`、`aria-label`、`role`、`label`、`placeholder`、按钮文本、表格列标题。
4. 识别 AntD 组件类型，例如 `Button`、`Input`、`Select`、`Table`、`Modal`、`Form.Item`。
5. 生成组件摘要和元素候选。

输出：

- `ui_component`
- `ui_element`
- `frontend_code` 向量文档
- 组件到页面、页面到路由的图关系

### 4.2 Playwright 运行态采集

静态代码只能知道“可能渲染”，不能保证运行时页面真实出现。需要用 Playwright 对关键路由和用户流采集运行态证据。

采集内容：

- 当前 URL。
- DOM 片段。
- accessibility snapshot。
- 全页截图和元素截图。
- 元素 bounding box。
- console error。
- network failure。
- 页面状态，例如权限、租户、feature flag。

运行态采集的价值：

- 发现动态渲染、权限控制、异步加载、国际化文案。
- 校验 locator 是否唯一、可见、可点击、可输入。
- 为视觉模型提供页面证据。

### 4.3 Java + BDD 自动化资产扫描

输入：

- `.feature` 文件。
- Cucumber step definition。
- Page Object 或 Screenplay task。
- Playwright Java 调用。

识别内容：

- Scenario、tags、业务域、数据依赖。
- Step 文本和对应 Java 方法。
- `page.locator(...)`、`getByRole(...)`、`getByText(...)`、CSS、XPath、正则 locator。
- action 类型：click、fill、select、hover、assert visible、assert text。
- locator 所在文件、行号、变量名、方法名。

输出：

- `test_asset`
- `step_element_binding`
- `locator_version`
- `bdd_steps` 向量文档

注意：MVP 可以先用 JavaParser + 规则解析常见调用；后续再引入 LLM 解析复杂封装。

### 4.4 执行结果回流

每次 CI 执行后，不只失败要回流，成功也要回流。

失败回流：

- 失败 locator。
- 异常类型，例如 timeout、strict mode、NoSuchElement、不可见、被遮挡。
- 失败截图。
- 当前 DOM 和 a11y tree。
- trace。
- 对应场景和 step。

成功回流：

- 当前 locator 仍然可用。
- 元素截图和 DOM signature 更新。
- locator 稳定分提升。
- 页面结构变化但脚本未失败的证据。

成功数据很重要，因为它能避免知识库只学习失败样本。

## 5. 自愈检索与决策流程

### 5.1 提交后主动影响分析

触发时机：

- 前端代码提交。
- Pull Request 创建或更新。
- merge 前 CI。

流程：

```text
读取 commit diff
  -> AST 解析变更组件
  -> 映射受影响页面和 UI 元素
  -> 图查询关联历史 BDD 场景
  -> 计算 locator 风险
  -> 选择最小回归测试集
  -> 执行自动化
  -> 更新知识库
```

风险评分因素：

- 元素文本、label、role、placeholder 是否变化。
- `data-testid` 是否新增、删除或重命名。
- AntD 组件类型是否变化，例如 `Input` 变成 `Select`。
- 元素父容器和列表结构是否变化。
- 该元素历史失败频率。
- 该 locator 是否是 CSS/XPath。
- 该元素是否属于高频核心业务流。

### 5.2 失败后响应式自愈

触发条件：

- Playwright Java 步骤因 locator 失败。
- strict mode 找到多个元素。
- 元素不可见、不可点击、超时。
- 断言失败但页面功能疑似可用。

流程：

```text
解析失败报告
  -> 定位失败 step 和旧 locator
  -> 从知识库召回目标元素历史画像
  -> 采集当前页面 DOM/a11y/screenshot
  -> LLM 生成候选 locator
  -> Playwright 实际验证候选
  -> 候选通过后生成 Java 代码补丁
  -> 重跑失败场景或受影响场景
  -> 保存 healing_event
  -> 自动提交 PR 或进入人工审批
```

### 5.3 候选 locator 验证规则

LLM 只能提出候选，不能直接决定修复成功。验证必须由 Playwright 和规则引擎完成。

硬性规则：

- locator 结果数量必须为 1，除非该步骤明确允许列表。
- 元素必须 visible。
- 对 click 行为，元素必须 enabled 且不被遮挡。
- 对 fill 行为，元素必须 editable。
- role/name/label 必须和目标意图一致。
- 候选 locator 不得使用过长绝对 XPath。
- 候选 locator 不得依赖随机 class、动态 id、序号 nth，除非没有替代方案且人工审批。

加分项：

- 使用 `data-testid`。
- 使用 `getByRole` + accessible name。
- 使用 `getByLabel`。
- 和历史元素截图或布局位置相近。
- 位于同一业务容器或相同表单项内。

### 5.4 置信度模型

建议把置信度拆成多个可解释分数：

```text
final_score =
  0.30 * semantic_score
+ 0.25 * runtime_validation_score
+ 0.15 * visual_score
+ 0.15 * code_change_score
+ 0.10 * historical_score
+ 0.05 * llm_self_score
```

阈值建议：

- `>= 0.90`：可自动生成 PR，并重跑验证。
- `0.75 - 0.90`：生成修复建议，人工审批后合并。
- `0.60 - 0.75`：只记录候选，不自动改代码。
- `< 0.60`：不自愈，保留失败。

## 6. 支持新故事卡的用例生成

新故事卡进入后，知识库应新增以下链路：

```text
StoryCard
  -> Requirement / Acceptance Criteria
  -> Frontend Diff / New Route / New Component
  -> Runtime Page Snapshot
  -> Candidate UI Elements
  -> Historical Similar Scenarios
  -> Generated BDD Scenarios
  -> Generated Java Steps / Page Objects
  -> Human Review
  -> Execution Feedback
```

生成策略：

1. 先基于验收标准生成 BDD 场景，不直接写 Java 代码。
2. 从知识库召回相似历史场景，例如“搜索 + 表格校验”、“新增表单 + 弹窗确认”、“权限控制”。
3. 将新页面运行态元素映射为 `element_uid`。
4. 生成 step 时优先复用已有 step definition。
5. 只有无法复用时才新增 Java step 或 Page Object 方法。
6. 生成后必须执行 dry-run 和真实浏览器验证。
7. 人类 review 后才固化到主分支。

## 7. Prompt 与输出契约

自愈 agent 的 LLM 输出必须是结构化 JSON，不允许自由文本驱动代码修改。

示例：

```json
{
  "target_intent": "点击登录按钮",
  "old_locator": "page.locator(\"#loginBtn\")",
  "failure_type": "NoSuchElement",
  "candidates": [
    {
      "locator_type": "role",
      "playwright_java": "page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName(\"Login\"))",
      "reason": "当前页面存在 role=button 且 accessible name 为 Login 的可见元素",
      "risk": "low",
      "confidence": 0.91
    }
  ],
  "requires_human_review": false
}
```

代码补丁 agent 的输出：

```json
{
  "file": "src/test/java/.../LoginPage.java",
  "old_code": "page.locator(\"#loginBtn\")",
  "new_code": "page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName(\"Login\"))",
  "patch_type": "locator_update_only",
  "tests_to_rerun": ["Login.feature:12"]
}
```

## 8. 技术选型建议

| 能力 | MVP 推荐 | 后续增强 |
| --- | --- | --- |
| 编排 | Python + LangGraph | 多 agent 工作流、人工审批节点 |
| LLM | GPT-5.2 做推理和代码修复 | GPT-4o/视觉模型处理截图，必要时本地 VLM |
| 浏览器采集 | Playwright Python 或 Playwright Java trace | Playwright MCP 作为 agent 工具层 |
| Java 代码解析 | JavaParser 或 tree-sitter-java | LLM 辅助复杂封装识别 |
| React 代码解析 | tree-sitter-typescript / Babel parser | 组件依赖图、路由图、AntD 语义增强 |
| 主数据库 | PostgreSQL | 加 CDC、审计、权限隔离 |
| 图关系 | Neo4j 或 AGE | 跨系统影响分析 |
| 向量库 | Qdrant / OpenSearch Vector | 混合检索和 rerank |
| 对象存储 | S3 兼容存储 | 截图相似度索引 |
| CI 集成 | Jenkins/GitHub Actions/GitLab CI webhook | 按风险动态选择回归集 |

## 9. 数据治理与安全

金融企业场景下需要默认考虑以下约束：

- 所有截图、DOM、trace 可能包含客户数据，进入模型前需要脱敏。
- LLM 调用必须记录 prompt、输入 hash、输出 hash、模型版本、操作者、审批人。
- 生产数据禁止直接进入训练集或长期向量库，除非经过合规审批。
- 自动修改测试代码必须通过 PR，不建议直接 push。
- 所有自动修复必须可以回滚。
- 高风险业务流，例如支付、交易、授权审批，默认人工审批。

## 10. MVP 范围建议

第一阶段不要试图一次性扫描全部系统。建议选择 1 到 2 个页面、20 到 50 个历史场景做闭环。

MVP 交付边界：

- 能扫描 Java BDD 项目中的 locator。
- 能采集一个前端页面的 DOM、a11y tree、截图。
- 能解析一次失败报告。
- 能召回历史元素画像。
- 能生成候选 locator。
- 能用 Playwright 验证候选 locator。
- 能生成修复 diff。
- 能重跑失败场景。
- 能保存 healing report。

MVP 暂不做：

- 全量自动合并。
- 全系统故事卡自动生成。
- 复杂跨页面流程规划。
- 端到端无人工审批。

## 11. 结合 UI-Root 的知识库完善方案

`UI-Root.html` 中的知识库节点不是独立向量库，而是贯穿触发、决策、自愈、执行和记录层的长期记忆中心。建议把知识库拆成三类能力：可追溯图谱、运行证据仓库、RAG 案例召回。

### 11.1 必须沉淀的链路

核心链路应从“前端提交”一直连到“测试修复结果”：

```text
commit_sha
  -> changed_file
  -> route/page
  -> React component
  -> AntD component / UI element
  -> locator candidate / locator version
  -> Page Object method
  -> BDD step
  -> scenario
  -> execution run
  -> failure diagnosis
  -> patch candidate
  -> review decision
```

这条链路用于回答三个问题：

1. 本次 PR 可能影响哪些 UI 元素和历史 BDD 场景。
2. 某个 locator 失败时，它对应的业务意图、历史截图、前端组件和过往修复是什么。
3. 自动补丁是否有证据支撑、是否可回滚、是否值得进入人工审批或自动 PR。

### 11.2 新增结构化表建议

在原有表基础上，建议补充以下表，直接对应 UI-Root 的记录层和知识库节点。

| 表 | 作用 |
| --- | --- |
| `project_binding` | 保存前端仓库、自动化仓库、CI pipeline、环境 URL、默认用户和权限集的绑定关系 |
| `route_map` | route、page_key、入口菜单、权限、feature flag 和源码文件的映射 |
| `component_graph_snapshot` | 每个 commit 下的 React/AntD 组件图快照，分 static/runtime 两类 |
| `locator_catalog` | 当前推荐 locator、历史 alias、稳定分、弃用原因和适用 locale |
| `test_impact_report` | 每次 PR 的受影响 routes、elements、scenarios、风险评分和选择的回归集合 |
| `healing_run` | LangGraph 每次运行的 trace_id、状态、分支、token 成本、耗时和最终结论 |
| `failure_diagnosis` | 失败根因、证据引用、L1/L2/L3 分级、建议动作和阻断原因 |
| `patch_candidate` | TSX testid 注入补丁、Java locator 补丁、等待策略补丁和验证结果 |
| `artifact_ref` | screenshot、trace、video、DOM、a11y、Allure 报告、diff 的 URI、hash 和 TTL |

### 11.3 Artifact 版本化规则

所有知识库 artifact 必须绑定 `commit_sha`、`trace_id`、`tool_version` 和 `source_repo`，否则后续无法解释和回滚。

```json
{
  "artifact_id": "artifact-20260427-001",
  "trace_id": "heal-run-001",
  "commit_sha": "abc123",
  "artifact_type": "locator_catalog",
  "uri": "s3://ui-kb/catalog/abc123/locator_catalog.json",
  "sha256": "...",
  "tool_version": "frontend-scanner@0.1.0",
  "created_at": "2026-04-27T16:00:00+08:00"
}
```

### 11.4 RAG 召回分层

自愈时不要只做一类相似度检索。建议按场景并行召回，再统一 rerank：

| 召回通道 | 输入 | 输出 | 用途 |
| --- | --- | --- | --- |
| 结构化精确召回 | old locator、POM 文件、行号、element_uid | 历史元素画像 | 首选，避免语义漂移 |
| 图谱影响召回 | commit diff、component node、route | 受影响 scenario 和 element | PR 影响分析 |
| 失败案例召回 | error_type、stack、step_text、trace 摘要 | 相似失败和修复案例 | Few-shot 修复参考 |
| 元素语义召回 | role、label、placeholder、nearby text | 候选等价元素 | locator 替换 |
| 视觉证据召回 | element crop、bbox、页面截图描述 | 历史视觉相似元素 | DOM 语义不足时兜底 |

### 11.5 L1/L2/L3 分级入库规则

知识库不能无差别吸收所有 AI 输出。建议使用以下准入规则：

- L1 自动修复成功并重跑通过：写入主知识库，提高 locator 稳定分，记录 patch。
- L1 候选通过但未重跑：只写入候选库，不作为成功经验。
- L2 人工审批通过：写入主知识库，并记录人工修改点。
- L2 人工驳回：写入负样本，后续 RAG 召回时作为反例。
- L3 环境或数据问题：写入故障分类库，不进入 locator 成功案例。

### 11.6 与前端 testid 注入的关系

UI-Root 中的 `Patch Generator` 同时承担 Java 补丁和前端 testid 增量注入。知识库应能反向输出“缺少稳定测试契约”的 PR 评论：

```text
页面: /trade/search
元素: 交易对手方输入框
风险: 当前历史用例依赖 placeholder，且本次 PR 修改了文案
建议: 为核心业务输入框补充 data-testid="trade-search-counterparty-input"
影响用例: Search trade by counterparty, Reset trade search form
```

这能把自愈从“被动修脚本”推进到“主动改善前端可测性”。

### 11.7 知识库验收标准补充

在 MVP 后，应增加以下验收项：

- 能从任意一个 Java locator 反查到 BDD step、scenario、页面、前端组件和最近一次执行证据。
- 能从任意一个 PR diff 查询受影响的 route、element、scenario 和推荐回归集合。
- 能区分成功自愈、失败自愈、人工驳回、环境故障四类历史样本。
- 能按 `commit_sha` 回放某次运行使用的 `route_map`、`component_graph`、`locator_catalog` 和 prompt 输入摘要。
- 能输出前端可测性风险清单，包括缺失 testid、依赖文案、依赖 CSS/XPath、portal scope 不明确等。
