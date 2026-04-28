# BEST_PRACTICES_ALIGNMENT

## Referenced Sources
- OpenAI Codex best practices:
  `https://developers.openai.com/codex/learn/best-practices`
- Claude Code best practices (zh-CN):
  `https://code.claude.com/docs/zh-CN/best-practices`

Reference check date: 2026-04-28.

## Applied Principles in This Repository
1. Clear task framing:
   implemented via `AGENTS.md` prompt structure (goal/context/constraints/done-when).
2. Repository guidance map:
   implemented via `AGENT.md`, `AGENTS.md`, `REVIEW_INDEX.md`.
3. Skills for repeated workflows:
   implemented via `SKILLS.md` and `.codex/skills/self-healing-ui/*`.
4. Small, verifiable increments:
   implemented via phase-by-phase docs in `phases/` and verification gates in `verification/`.
5. Evidence-first completion:
   enforced through artifact requirements in verification files and templates.
6. Human-in-the-loop risk gates:
   enforced through L1/L2/L3 routing in plan, strategy, and config samples.
7. Structured project memory:
   implemented via config templates, phase artifacts, and review templates.
