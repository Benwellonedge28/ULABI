# ULABI Enumeration Design

- **Status** · Proposed implementation design
- **Specification baseline** · [ULABI Core 0.1.0 draft](../../ULABI-SPEC.md)
- **Integration** · [Data types](../abi/data-types.md), [Tagged unions](unions.md), [Versioning](../../ULABI-VERSIONING.md)

## Scope

Define scalar enum values separately from union variants carrying payloads.

## Contract

- Record the backing integer width, signedness, representation and stable mapping from named variant IDs to integer values. Reject duplicate values unless the descriptor explicitly defines aliases to the same semantic value.
- Decide whether the enum is closed or explicitly extensible. For an extensible enum, preserve an unknown numeric value only within the declared range and expose it distinctly from known variants.
- Preserve existing assigned values across compatible revisions. A new variant in a closed enum requires compatibility analysis for older receivers.

## Implementation sequence

1. Validate the enum descriptor against its integer range and generate explicit conversion tables for both directions.
2. Decode the integer using the selected representation, then validate the range and variant rule before creating a language enum value.

## Acceptance criteria

| Case | Expected result |
| --- | --- |
| Native language uses a wider enum repr | Convert explicitly to the declared ULABI width. |
| Unknown value in closed enum | Return a typed conversion error. |
| Known variant's numeric value changes | Report a compatibility break. |
