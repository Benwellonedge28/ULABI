# ULABI Embedded and Bare-Metal Profile Design

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Resource limits](../runtime/resource-management.md), [Architecture mappings](architectures.md), [Memory allocation](../memory/allocation.md)

## Scope

Define a constrained deployment with explicit memory, device, timing and startup contracts.

## Contract

- Publish flash and RAM budgets, static or bounded allocation policy, stack bounds, target memory map and available peripherals. A bare-metal host may use statically linked bindings and an approved initialization table.
- Declare supported call contexts, especially interrupt versus ordinary execution. Restrict interrupt-bound operations to the declared bounded behavior. Do not promise a deadline unless the timing profile provides evidence.
- Expose optional features through discovery. Unsupported host services return the mapping’s defined failure. Memory-mapped I/O requires a declared access grant, width, alignment and ordering.

## Implementation sequence

1. Generate the allocation and export tables at build time where possible. Verify against target limits and validate map and platform identity at startup.
2. Exercise static fixtures on an emulator and validate hardware-sensitive behavior on the declared target. Measure stack and allocation bounds for the release configuration.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Static buffers exceed the profile capacity | Fail the build or admission before execution. |
| Blocking service invoked from forbidden interrupt context | Reject before blocking or entering the service. |
| No dynamic loader exists | Use the declared static binding or report the profile unsupported. |
