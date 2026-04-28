# Research Notes（中文）

本文档是 `research-notes.md` 的中文配套版本，用于 AI 辅助 UI 自愈测试建议时的参考。

## Playwright
- 优先使用内置 locator：role、text、label、placeholder、alt text、title、test id。
- role locator 应尽量带 accessible name。
- test id 适合作为稳定测试契约，尤其用于核心业务和多语言场景。
- strictness 多匹配错误是定位范围不清的信号，不建议默认 `first()`。
- Playwright MCP 的 a11y 快照可提供结构化页面语义，比仅看截图更可靠。

## Healenium
- Healenium 是 Selenium 生态最接近 locator 自愈的开源参考。
- 可复用其思路：历史选择器、拦截修复、修复报告、选择器历史、评分。
- 不应直接照搬到 Java Playwright；应由 Playwright 运行验证和 Java 补丁实现落地。

## Repo Map / AST
- Repo map 通过 AST/Tree-sitter 提取符号关系。
- 本项目建议扩展到：
  - React 路由/组件图
  - Ant Design 组件关系图
  - Java BDD feature/step/Page Object/locator 图
  - 提交影响图

## 设计偏好
- 候选是否有效由规则与 Playwright 决定。
- LLM 负责候选提议、意图解释、补丁草案。
- 知识库存事实、版本与证据。
- 向量库用于相似元素/步骤/失败/修复召回。
- 业务语义、断言语义和高风险流程保持人工审查。
