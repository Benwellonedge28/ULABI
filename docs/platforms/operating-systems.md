# ULABI Operating System Integration

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Architecture mappings](architectures.md), [Runtime boundary](../runtime/runtime-interface.md), [Loader](../tooling/loader-interface.md)

## Scope

Define OS-facing loader, lifecycle and service facts that a host must supply to a selected mapping.

## Contract

- Declare object format, OS and runtime compatibility, native ABI, loader service, thread affinity and process capabilities. Match a component against these facts before execution.
- Route files, devices, clocks and network access through declared host services. Translate host failures into the operation’s ABI errors while retaining authorized diagnostic context.
- Define teardown and resource ownership at the OS boundary. Scope native handles to the issuing host and lifetime, and provide explicit transfer semantics where supported.

## Implementation sequence

1. Build an OS mapping adapter for module open, prepare, bind and close. Pair it with the supported architecture and verify the exact supported configuration.
2. Exercise missing service, unavailable OS revision, process shutdown and handle cleanup. Surface absent optional controls through capability discovery.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Module format mismatches the host | Reject before load. |
| Native handle found in a cross-host message | Reject unless a declared service resolves a transferable identity. |
| Host capability is absent | Apply required/optional behavior from the profile. |
