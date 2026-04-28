# SKILLS.md

## Purpose
Define project-level skills for repeatable workflows in UI self-healing testing.

## Active Skill
- `self-healing-ui`
  - Path: `.codex/skills/self-healing-ui/SKILL.md`
  - Scope: Java + BDD + Playwright failure healing, runtime validation, patch governance, and knowledge-loop planning.

## Recommended Skill Portfolio
1. `self-healing-ui` (already present)
2. `frontend-impact-analysis`
3. `java-patch-safety-review`
4. `phase-gate-verifier`
5. `artifact-sanitizer`

## Skill Contract Template
Each skill should explicitly define:
- What job it handles
- When to invoke it
- Required inputs
- Expected outputs
- Stop conditions and escalation paths

## Skill Quality Rules
- One skill, one bounded job.
- Start with 2-3 concrete use cases.
- Prefer deterministic steps and structured output.
- Use scripts/assets only when reliability improves.
- Move repeated prompt patterns into skills.

## Versioning
- Keep skill specs under source control.
- Record behavioral changes in commit messages.
- Keep shared skills in repo; keep personal variants local.
