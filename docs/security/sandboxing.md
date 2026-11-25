# ULABI Sandbox Boundaries

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Security model](security-model.md), [Process model](../runtime/process-model.md), [Resource management](../runtime/resource-management.md)

## Scope

Define isolation selection and enforcement for components with limited trust or resource scope.

## Contract

- A sandbox profile lists allowed host services, filesystem and network access, resource limits and handle inheritance. Select an isolation mechanism the host can enforce before the component runs.
- Use a mediated IPC or runtime boundary for components that cannot execute inside the host's trust boundary. Expose only declared entry points and approved capabilities. Treat shared memory as an explicit grant.
- On isolation failure or teardown, stop new admissions, revoke resource access and reconcile outstanding work under the process model.

## Implementation sequence

1. Evaluate host support for every mandatory restriction. Prepare a minimal execution context and check that no unintended handles are inherited.
2. Start the component inside the enforcement boundary, then verify readiness and limits. Record the effective sandbox scope for audit.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Required network restriction is unavailable | Refuse to start under that profile. |
| Component requests a service outside the allowlist | Block the request at the host boundary. |
| A worker is terminated | Release resources and invalidate outstanding handles. |
