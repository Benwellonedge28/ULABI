# ULABI Boundary Debugger Interface

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Deterministic debugging](../observability/deterministic-debugging.md), [Type descriptors](../type-system/type-descriptors.md), [Authorization](../security/authorization.md)

## Scope

Define authorized inspection of component boundaries without relying on language-specific object layouts.

## Contract

- Bind inspection to an authorized component instance and a pinned mapping. Resolve symbol, source and type data using exact artifact and debug-information identities.
- Expose declared lifecycle, boundary call, typed metadata and diagnostic context. A pause or snapshot is explicit and only available where the runtime and profile support it. Apply redaction before export.
- Return an unknown or unsupported state when information is absent. Do not infer live memory ownership or follow arbitrary pointers from a descriptor.

## Implementation sequence

1. Authorize and establish a bounded inspection session. Attach to the current instance generation and select a supported observation mode.
2. Read descriptor-backed state while respecting runtime pause and lifetime rules. Release the snapshot and invalidate the session on generation changes.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Debug information belongs to another build | Reject symbol resolution for that artifact. |
| Inspect permission excludes arguments | Show allowed metadata and redact the argument fields. |
| Instance restarts mid-session | Close or refresh against the new generation explicitly. |
