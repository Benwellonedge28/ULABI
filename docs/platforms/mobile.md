# ULABI Mobile Host Profile

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [OS integration](operating-systems.md), [Runtime lifecycle](../runtime/runtime-interface.md), [Authorization](../security/authorization.md)

## Scope

Define ABI bindings under mobile process lifecycle, packaging constraints, and OS permissions.

## Contract

- Publish the target mapping per packaged architecture and the minimum compatible host version. Use the loading and signing constraints imposed by the selected host policy.
- Distinguish foreground, suspended, background and terminated host conditions. Admission of a background operation depends on declared host service availability and permission.
- Persist only committed state using the declared durable interface. Release short-lived buffers, UI dispatch tokens and grants on teardown or generation change.

## Implementation sequence

1. Validate packaged binary slices and manifest against the host mapping. Bind permissions through the host and propagate denial or revocation explicitly.
2. Implement lifecycle callbacks that stop or resume admitted work as defined by the component, with recovery testing after process termination.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Process resumes after OS termination | Create a new instance and restore only committed compatible state. |
| Background execution grant is missing | Return the declared unavailable or policy status. |
| Wrong architecture binary is packaged | Reject at mapping validation. |
