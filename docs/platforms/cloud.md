# ULABI Cloud Deployment Profile

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Server profile](server.md), [Distributed ABI](../distributed/distributed-abi.md), [Supply-chain evidence](../security/supply-chain-security.md)

## Scope

Define deployment-level compatibility, isolation and routing across independent hosts.

## Contract

- Declare each deployed artifact digest, target mapping, transport, required capabilities, isolation policy and compatible peer versions. Bind deployment instances to those identities before routing.
- Use explicit grants and quotas scoped to a workload or tenant. Resolve placement against verified host and accelerator capabilities. Keep credentials and region or placement policy under host control.
- Coordinate upgrades with pinned compatibility matrices and readiness checks. Keep remote delivery uncertainty distinct from local binding failure and restrict retries to declared safe operations.

## Implementation sequence

1. Validate artifacts and profile compatibility during deployment admission. Route across versions only when the negotiated contract is supported in that direction.
2. Test partial rollout, zone or host loss and state recovery. Confirm new instances preserve identity, isolation, and contract selection.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Two versions share a fleet with no compatible remote contract | Keep them from exchanging requests. |
| Scheduler offers mismatched hardware | Reject placement and report the missing capability. |
| Endpoint outcome is uncertain after failover | Apply declared reconciliation or report uncertainty. |
