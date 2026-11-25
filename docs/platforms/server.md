# ULABI Server Host Profile

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Process model](../runtime/process-model.md), [Resource management](../runtime/resource-management.md), [Health monitoring](../reliability/health-monitoring.md)

## Scope

Define long-lived service component admission, request isolation, and deployment lifecycle.

## Contract

- Bind component versions to instance generations. Define readiness separately from startup and liveness. Route requests only to ready instances with the required role and profile.
- Scope work to a request or tenant with explicit budgets and grants. Propagate deadlines and classify timeout separately from operation failure when delivery state is uncertain.
- On replacement stop new admission, drain according to the declared limit, and close old handles. Replays require a declared idempotency or transaction contract.

## Implementation sequence

1. Stage a verified component version, initialize it, and check readiness before switching routing. Keep dependent mappings pinned through the rollout.
2. Observe saturation, readiness and completed work independently. Exercise graceful and forced shutdown with the same admission path.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| New generation fails readiness | Keep it out of routing. |
| One tenant exhausts its queue | Apply its limit without spending another tenant’s budget. |
| Old handle arrives after cutover | Reject it as stale. |
