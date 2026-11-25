# ULABI Desktop Host Profile

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [OS integration](operating-systems.md), [Thread affinity](../runtime/threading.md), [Capabilities](../security/capability-security.md)

## Scope

Define local desktop integration for a verified component, including UI dispatch and resource access.

## Contract

- Declare OS, architecture and loader mappings and whether an operation requires the main UI executor. Keep rendering toolkit and application internals behind adapters.
- Use host-mediated user selections and resource grants. Treat a selected file or device as a scoped resource with a declared validity period and explicit release.
- Define application quit behavior for pending work, UI callbacks and session resources. Persistent component state uses a versioned snapshot or host storage contract.

## Implementation sequence

1. Bind platform services during initialization and route UI-bound callbacks through the declared executor.
2. Test startup, conflicting OS mappings, grant revoke, background work, and quit while callbacks are pending.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Background worker requests UI-bound access | Dispatch or reject per metadata. |
| User revokes an open resource grant | Deny subsequent protected actions. |
| Application quits before work completes | Drain or cancel under the declared lifecycle and release resources. |
