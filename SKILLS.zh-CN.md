# SKILLS.md（中文）

## 目的
定义项目级 skill，用于 UI 自愈测试中的可重复工作流。

## 当前启用 Skill
- `self-healing-ui`
  - 路径：`.codex/skills/self-healing-ui/SKILL.md`
  - 范围：Java + BDD + Playwright 失败自愈、运行时验证、补丁治理与知识回流规划。

## 推荐 Skill 组合
1. `self-healing-ui`（已存在）
2. `frontend-impact-analysis`
3. `java-patch-safety-review`
4. `phase-gate-verifier`
5. `artifact-sanitizer`

## Skill 契约模板
每个 skill 需要明确：
- 处理哪类任务
- 何时触发
- 必需输入
- 预期输出
- 停止条件与升级路径

## Skill 质量规则
- 一个 skill 只负责一个边界清晰的工作。
- 先覆盖 2-3 个具体场景。
- 优先确定性步骤和结构化输出。
- 仅在提升可靠性时引入脚本/资产。
- 重复提示模式应沉淀为 skill。

## 版本管理
- skill 规范纳入版本控制。
- 行为变化写入提交说明。
- 团队共享 skill 保存在仓库，个人变体保存在本地。
