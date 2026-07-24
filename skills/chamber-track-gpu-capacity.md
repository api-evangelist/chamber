---
name: Track GPU capacity and spend
description: Read remaining GPU-hour budgets by pool and team, roll up workload statistics, and rank workloads by utilization to find idle GPUs with the Chamber API.
api: openapi/chamber-openapi.yml
operations: [getCapacity, getWorkloadStats, listRankedWorkloadMetrics]
---

# Track GPU capacity and spend

Use the Chamber API to understand how GPU capacity is allocated and consumed across an organization.

## Auth & headers
- `Authorization: Bearer <token>` (Cognito JWT or `ch.` API token) and `X-Organization-Id: <org-id>` on every request.
- Base URL: `https://api.usechamber.io/v1`.

## Steps
1. **Read the budget** — call `getCapacity` to get `summary.total_remaining_gpu_hours` and `current_active_gpus`, plus per-pool (`pool_id`) and per-team (`initiative_id`) allocated/used/remaining GPU hours. Narrow with the `initiative_id` or `pool_id` query filter.
2. **Roll up usage** — call `getWorkloadStats` with a `time_range` (`today`, `this_week`, `last_7_days`, `last_30_days`, or `custom` with `start_date`/`end_date`) and `group_by` (`submitted_by`, `initiative_id`, or `instance_type`) to get `total_gpu_hours`, `success_rate`, and grouped breakdowns.
3. **Find idle GPUs** — call `listRankedWorkloadMetrics` with `sort_by=gpu_utilization` and `order=asc` to surface the least-utilized workloads; compare each against `scope_average.gpu_utilization`.

## Conventions
- Responses wrap payloads as `{ data, metadata?, request_id }`; `getWorkloadStats` and `getCapacity` return a single `data` object (no pagination).
- Errors follow `{ error: { code, message } }` — see `errors/chamber-problem-types.yml`.
- GPU pricing for cost math is configured per-org in the dashboard, not returned by these endpoints (see `lifecycle/`/settings docs).
