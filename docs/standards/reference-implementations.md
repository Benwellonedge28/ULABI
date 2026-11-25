# ULABI Reference Implementations

**Status** · Proposed implementation design  
**First milestone** · Native interoperability pilot  
**Architecture baseline** · [ULABI design 0.2.0-draft](../../ULABI-DESIGN.md) and [Core specification 0.1.0 draft](../../ULABI-SPEC.md)

## Purpose

Build two independent implementations of the same small ULABI interface and prove that a binary built in either language can be consumed by the other. The pilot turns the repository’s existing ABI and FFI designs into executable evidence. Its results apply to the tested interface, profile revision and platform. Full Core conformance requires coverage beyond this pilot.

The [Core specification](../../ULABI-SPEC.md) governs the semantic contract. The pilot fixes a single platform mapping so that implementations can compare observable behavior. Any conflict between the proposed mapping and the Core specification must be resolved before freezing the pilot fixtures.

## Pilot scope

| Decision | Proposed implementation |
| --- | --- |
| Platform | Linux x86-64, shared libraries and the platform C calling convention |
| Language pairs | A C consumer with a Rust provider, and a Rust consumer with a C provider. Same-language pairs serve as controls. |
| Interface shape | One synchronous, in-process operation using a byte buffer, a fixed-width length and an explicit status result |
| Memory | Caller-supplied buffers. Borrowed inputs expire when the call returns. |
| Binding | A provider-specific, explicitly selected shared library and an accompanying interface manifest |
| Profile evolution | A later target supplies a separate platform mapping. The interface identity and operation semantics stay stable across mappings. |

The pilot covers local calls. Callback, asynchronous, distributed and accelerator profiles each require their own contract and conformance cases. Zamani and Sankofa can add independent adapters after the C and Rust exchange is reproducible.

## Interface and native mapping

Use a bounded byte-copy interface so that the first test exercises type mapping, lifetime, symbol binding and error behavior without relying on a particular language’s strings, allocator or exceptions.

| Contract element | Pilot choice |
| --- | --- |
| Interface identity | `org.ulabi.pilot.byte-copy` at interface version `1.0`. A fixture-local identifier pending any registry policy. |
| Operation | `copy_bytes` |
| Native binding | Status `u32` returned in the platform convention. Parameters in order are input address, input length `u64`, output address, output capacity `u64`, and address of the written length `u64`. |
| Native export | `ulabi_pilot_byte_copy_v1`, scoped to the selected library |
| Limits | Each declared length and capacity is at most 1 MiB. Providers accept calls independently of previous calls. |

A caller supplies valid, suitably aligned addresses for the declared regions. A null input is valid for zero input length. A null output is valid for zero capacity. The written-length address is mandatory. For a call that may write data, the input, output and written-length regions must be pairwise disjoint. The adapters must keep borrowed regions live through the call and must not retain addresses after it. Run the pilot with trusted binaries. Native foreign calls assume valid accessible addresses from their caller.

Providers validate observable argument conditions before touching either data buffer. Validation order is missing written-length address, invalid declared lengths or null data addresses, then insufficient output capacity. On success, the provider copies exactly the input length, updates the written length, and leaves unused output capacity unchanged. With empty input it sets the written length to zero and accesses neither data buffer. On insufficient capacity, it reports the required length through the written-length address and leaves the output buffer unchanged. Where the written-length address is valid, it sets written length to zero on other errors.

The fixture reserves four pilot status values. These are status codes for this operation and are mapped explicitly to each adapter’s error representation.

| Value | Status | Meaning |
| --- | --- | --- |
| 0 | Success | Output contains the copied bytes and written length equals input length. |
| 1 | Invalid argument | A required address is null, or a null data address has a nonzero declared extent. |
| 2 | Limit exceeded | A declared extent exceeds the pilot limit. |
| 3 | Insufficient capacity | The output cannot hold the input. Written length reports the required capacity. |

For the second validation step, check declared extents before validating data addresses so simultaneous failures produce the same status across implementations. Adapter internals must prevent language-level unwinding from crossing the ABI. The copy path should require no allocation or callbacks and use operations that cannot panic for inputs satisfying the stated caller contract.

## Manifest and binding

Define the manifest schema as a pilot fixture with a revision, ULABI Core baseline, interface ID and version, platform mapping ID, operation signature and export name. Keep the semantic interface version separate from the manifest schema revision and platform mapping ID. Use a trusted fixture manifest paired with each provider binary through the build output. Validate every required field against the consumer’s fixture before loading the selected library or invoking code. Initially require exact matches for the profile revision and interface version. Report metadata errors, unsupported mappings and missing exports separately from operation statuses. A rejected manifest must cause zero calls into the provider.

Keep the C declaration and Rust foreign signature independently authored against the same fixture contract. The harness compares their observable behavior rather than sharing an adapter implementation. Bind the symbol on the selected library handle and retain that handle for the lifetime of the binding. Each consumer runs against one provider artifact at a time. Build providers before consumers run and record the provider digests used by the harness.

## Proposed implementation layout

| Path | Responsibility |
| --- | --- |
| `reference/native-pilot/fixtures/` | Versioned interface manifest, public calling contract and expected outcomes independent of either implementation |
| `reference/native-pilot/c/` | C provider, C consumer and their native declaration |
| `reference/native-pilot/rust/` | Rust provider, Rust consumer and their independently authored native declaration |
| `reference/native-pilot/tests/` | Matrix runner, fixture validation and result collection |

Use one local entry point to build both providers, build both consumers and run all four pairings. The runner accepts provider artifact paths and never silently substitutes a same-language provider when a cross-language test fails. Give each case a stable ID and record its outcome, the consumer and provider identities, the manifest revision, target and compiler versions. Execute the suite offline with the tools installed and emit machine-readable results alongside a short human-readable summary.

## Verification and release gate

| Case | Expected evidence |
| --- | --- |
| Required positive paths | All four pairings handle empty input, bytes containing zero and values above `0x7f`, exact capacity and a payload at the size limit. Output matches byte for byte. |
| Buffer semantics | Excess output capacity retains its sentinel values. Insufficient capacity returns the required length and leaves every output byte unchanged. |
| Input validation | Null addresses with nonzero extents, null written-length address and limit-plus-one extents produce the prescribed status in each pairing. Cases with overlapping failures confirm evaluation order. Test inputs use valid storage except for documented null cases. |
| Binding failures | Wrong interface identity, version, platform mapping or signature fails before provider invocation. A compatible fixture paired with a test library that omits the export yields a separate binding error. |
| Binary independence | Each consumer runs against the other compiler’s prebuilt provider with identical expected results. The runner checks the intended provider artifact was used for each pairing. |
| Repeatability | A clean offline build reproduces all expected outcomes with the recorded toolchain versions. |

Freeze the pilot fixture only after comparing it with the [calling convention](../abi/calling-convention.md), [data types](../abi/data-types.md), [memory model](../abi/memory-model.md), [exception model](../abi/exception-model.md), and [FFI](../interoperability/foreign-function-interface.md) designs. Implement and verify the two providers independently, then add the two consumers and run the full matrix. Publish the fixture revision and results together. Use the matrix as input to the future [conformance policy](conformance.md) and [test suite](test-suite.md).

A second platform should reuse the semantic fixture and add its own native declaration and test matrix. A new language should first implement one side against an existing provider or consumer, then run the reverse pairing before claiming support for both roles.
