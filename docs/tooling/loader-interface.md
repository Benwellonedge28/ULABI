# ULABI Loader Interface

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Secure loading](../security/secure-loading.md), [Runtime lifecycle](../runtime/runtime-interface.md), [Linker interface](linker-interface.md)

## Scope

Define runtime prepare, open, bind and close operations for verified components.

## Contract

- Accept an exact artifact identity, manifest and approved policy. Return an instance-scoped handle or a structured validation, policy, load, binding or initialization failure.
- Separate prepare from execution. Finish compatibility and security admission for artifacts and required dependencies before an open that can run code. Bind by the selected instance’s handle.
- Track outstanding bindings and borrowed resources. Close blocks, drains or reports busy according to the declared lifecycle. Invalidate handles only with generation and ownership semantics preserved.

## Implementation sequence

1. Prepare a verified artifact plan, open that exact artifact, resolve its required exports, and initialize the component before publishing the instance.
2. On failure, unwind acquired resources in reverse order. Test admission and teardown failures without leaving a partially published instance.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Two libraries export the same native symbol | Resolve it only on the admitted provider handle. |
| Initialization fails after bind | Close or clean up and expose no ready handle. |
| Outstanding async work may still execute code | Keep the module loaded until the lifecycle contract permits close. |
