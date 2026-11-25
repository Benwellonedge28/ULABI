# ULABI Accelerator Platform Integration

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [GPU hardware](../hardware/gpu.md), [Quantum hardware](../hardware/quantum.md), [Async model](../runtime/async-model.md), [Capability discovery](../compatibility/capability-discovery.md)

## Scope

Define how a host discovers, binds and schedules optional accelerator providers. The hardware designs own individual execution models.

## Contract

- Describe device identity, supported operations, mappings, buffer locations, alignment, limits and synchronization. Select against declared capabilities and a scoped device grant.
- Represent submissions with job and buffer handles. Carry placement and ownership explicitly and keep buffers valid until the completion contract releases them.
- Define transfer, completion, cancellation and device-loss outcomes before admission. Use a fallback only where explicitly permitted and semantically compatible. Report which execution target ran.

## Implementation sequence

1. Discover the provider and validate its capability snapshot. Reserve device and memory capacity before staging buffers and submitting work.
2. Observe completion and fences, settle result ownership, then release reservations. On device loss, invalidate affected handles and route safe recovery through policy.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Device does not support the requested data layout | Reject or use the declared conversion. |
| Device is lost after submission | Report job disposition and invalidate its handles. |
| Fallback has weaker precision than promised | Reject fallback. |
