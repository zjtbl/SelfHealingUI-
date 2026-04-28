# V1 LangGraph State Machine

Updated: 2026-04-28  
Timezone baseline: `Asia/Shanghai`

This is the English companion of [V1_LANGGRAPH_STATE_MACHINE.zh-CN.md](./V1_LANGGRAPH_STATE_MACHINE.zh-CN.md).

## Scope

1. Jenkins webhook -> failure analysis -> decision loop (V1).
2. Optional locator healing write-flow (V1.5 feature flag).

## Decision Paths

1. `NO_HEAL`
2. `HEAL_CANDIDATE` (analyze-only in V1)
3. `NEED_HUMAN`

## Optional Write-Flow (V1.5)

1. create heal branch from latest `main`
2. commit patch
3. create Draft MR
4. trigger branch CI rerun
5. on rerun failure, revert commit
6. on revert conflict, mark and stop
