# V1 API Contract

Updated: 2026-04-28  
Timezone baseline: `Asia/Shanghai`

This document defines V1 platform APIs for:

1. Jenkins webhook ingestion
2. Platform-triggered Jenkins runs
3. Event/run/decision query
4. MR change display
5. Retry and revert controls

Primary reference version is [V1_API_CONTRACT.zh-CN.md](./V1_API_CONTRACT.zh-CN.md).

## Core Endpoints

1. `POST /api/v1/webhooks/jenkins`
2. `POST /api/v1/projects/{project_id}/jobs/{job_id}/trigger`
3. `GET /api/v1/events/{event_id}`
4. `GET /api/v1/runs/{run_id}`
5. `GET /api/v1/runs/{run_id}/mr-changes`
6. `POST /api/v1/runs/{run_id}/retry`
7. `POST /api/v1/runs/{run_id}/revert`

## V1 Rules

1. Accept partial webhook payload and enrich context from Jenkins APIs.
2. Keep idempotency by payload hash and optional `X-Idempotency-Key`.
3. Locator-only healing scope for V1.
4. No automatic merge to `main`.
5. Revert conflict must be marked and stopped.
