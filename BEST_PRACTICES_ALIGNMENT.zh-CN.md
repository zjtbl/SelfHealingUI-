# BEST_PRACTICES_ALIGNMENT（中文）

## 参考来源
- OpenAI Codex 最佳实践：
  `https://developers.openai.com/codex/learn/best-practices`
- Claude Code 最佳实践（中文）：
  `https://code.claude.com/docs/zh-CN/best-practices`

参考核对日期：2026-04-28。

## 本仓库已落地的实践映射
1. 明确任务边界：
   通过 `AGENTS.md` 中的 goal/context/constraints/done-when 结构落地。
2. 仓库导航地图：
   通过 `AGENT.md`、`AGENTS.md`、`REVIEW_INDEX.md` 落地。
3. 重复流程 skill 化：
   通过 `SKILLS.md` 与 `.codex/skills/self-healing-ui/*` 落地。
4. 小步可验证推进：
   通过 `phases/` 分阶段说明与 `verification/` 验证门禁落地。
5. 证据优先完成定义：
   通过阶段验证文件与模板中产物要求落地。
6. 人在回路风险门禁：
   通过 L1/L2/L3 分流在计划、策略与配置样例中落地。
7. 结构化项目记忆：
   通过配置模板、阶段产物与评审模板落地。
