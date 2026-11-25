# ULABI Runtime Boundary

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Core ABI](../abi/core-abi.md), [Loader](../tooling/loader-interface.md), [Resource management](resource-management.md)

## Scope

Define the lifecycle and service surface through which a host prepares and runs a verified component.

## Contract

- Use a versioned host descriptor with operation identities, role and mapping, capabilities, and lifecycle state. Resolve required services before initialization. Expose only the services granted by the selected profile and loader policy.
- Model the normal progression as Discovered, Verified, Loaded, Initialized, Ready, Running, Stopped, Unloaded. Declare which pause, degradation and recovery transitions a profile supports. Reject operation calls outside admitted states.
- Handle failed initialization transactionally. Reject new work when stopping, drain or cancel according to the contract, and release handles and borrows before unloading executable code.

## Implementation sequence

1. Bind a verified component to a host context and validate versions, limits and required services. Publish Ready only after initialization succeeds.
2. Track state transitions and admitted work by component generation. Coordinate stop with async operations and prevent stale handles from addressing a replacement.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Required host service unavailable | Fail initialization before the component becomes callable. |
| Unload races an admitted call | Wait or report busy according to policy and keep code loaded until the call ends. |
| Init fails after partial setup | Release acquired resources and record the initialization failure. |
