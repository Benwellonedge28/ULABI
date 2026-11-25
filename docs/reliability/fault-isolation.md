# ULABI Fault Isolation

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Fault detection](fault-detection.md), [Sandboxing](../security/sandboxing.md), [Recovery](recovery.md)

## Scope

Limit the impact of a detected fault to the smallest enforceable failure domain.

## Contract

- Select a declared failure domain, such as component, process, resource or service. Record affected generations, operations and dependents, and identify what isolation the host can enforce.
- Stop new admissions and revoke scoped access where policy requires it. Drain, cancel or reconcile admitted work by its operation contract. Coordinate shared-resource cleanup with ownership.
- Escalate if a mandatory isolation boundary cannot be enforced. Maintain auditability and preserve unaffected components and grants.

## Implementation sequence

1. Map evidence to a domain and obtain the policy decision. Quarantine the affected instances and confirm the isolation controls took effect.
2. Record in-flight work and its outcome uncertainty before handing the incident to recovery or escalation.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| One tenant’s worker faults | Quarantine its failure domain without stopping independent tenants. |
| In-flight write has unknown outcome | Record it for reconciliation rather than assuming failure. |
| Selected boundary cannot block new admission | Escalate before automated recovery. |
