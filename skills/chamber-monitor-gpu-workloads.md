---
name: Monitor GPU workloads
description: Find running GPU workloads, inspect one, and check its GPU utilization/memory/temperature/power metrics using the Chamber API.
api: openapi/chamber-openapi.yml
operations: [listWorkloads, getWorkload, getWorkloadMetrics]
---

# Monitor GPU workloads

Use the Chamber API to observe GPU workloads for an organization.

## Auth & headers
- Send `Authorization: Bearer <token>` (a Cognito JWT or a Chamber API token prefixed `ch.`).
- Send `X-Organization-Id: <org-id>` on every request (required except `/v1/health`).
- Optionally send `X-Request-Id` for tracing; it is echoed back as `request_id`.
- Base URL: `https://api.usechamber.io/v1`.

## Steps
1. **List active workloads** — call `listWorkloads` with `status=running` and a `limit` (1-100, default 20). Page with `next_token` from `metadata.next_token` until it is absent. Optionally filter by `initiative_id`, `submitted_by`, or a `start_date`/`end_date` window.
2. **Inspect one** — take a `workload_id` (prefix `wl_`) from the list and call `getWorkload` to read its status, `instance_type`, `gpu_count`, timestamps, and `gpu_hours_used`.
3. **Check its metrics** — call `getWorkloadMetrics` for that `workload_id` with a `time_range` (`last_1h`, `last_6h`, `last_24h`, `job_lifetime`) and a comma-separated `metrics` list (`gpu_utilization,memory_utilization,temperature,power_usage`).

## Conventions
- Responses wrap payloads as `{ data, metadata?, request_id }`.
- Errors return `{ error: { code, message, details? }, request_id }` — see `errors/chamber-problem-types.yml`. A `403` means your role (`ORG_ADMIN`/`TEAM_LEAD`/`MEMBER`) or team scope does not permit access; a `404` means the workload id does not exist in the org.
- The API is read-only; there is no idempotency-key contract (see `conventions/chamber-conventions.yml`).
