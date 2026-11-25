# ULABI Test Suite Design

**Status** · Proposed implementation design  
**First milestone** · Native C/Rust interoperability pilot  
**Design dependencies** · [Core specification 0.1.0 draft](../../ULABI-SPEC.md), [reference implementations](reference-implementations.md), [calling convention](../abi/calling-convention.md), [FFI](../interoperability/foreign-function-interface.md)

## Purpose and scope

Build an offline test suite that verifies the first ULABI native interface on Linux x86-64. It runs each C or Rust consumer against each independently built C or Rust shared library. Both cross-language directions are required. The same-language pairings help localize failures.

The [reference implementation design](reference-implementations.md) defines the pilot's byte-copy semantics. This document defines the fixtures, execution order, evidence, and release gate for those semantics. Passing the suite demonstrates interoperability for the tested interface, platform mapping, and revisions. Full Core conformance needs its own requirement set and evidence under the future [conformance policy](conformance.md).

## Repository layout and ownership

| Future path under `reference/native-pilot/` | What to implement |
| --- | --- |
| `fixtures/manifest-v1.json` | Canonical pilot manifest. Stage identical content with each successful provider build. |
| `fixtures/cases-v1.json` | Ordered case table, expected binding outcomes, operation status, written length, output expectations, and provider artifact role. Author expectations from the contract. |
| `c/` and `rust/` | Independent provider libraries and consumer executables. Each language also builds a missing-export fixture library. Each consumer owns its own native declaration and manifest validation. |
| `tests/run.py` | Suite entry point using the Python standard library. Coordinates builds, runs, deadlines, comparison, and reporting. |
| `tests/test_runner.py` | Runner unit tests using temporary directories and small fake child processes to exercise failure handling and report validation. |
| `tests/toolchains.json` | Tested tool versions and commands for the pilot target. Pin exact resolved versions before the first release gate. |

Retain the C and Rust declaration files as separate sources. Share fixture data, test inputs, expected outcomes, and result schemas. Provider and consumer code should build independently from the other language's output directory.

## Manifest fixture and binding

Use UTF-8 JSON with exactly the fields below in revision 1. Treat object key order as immaterial. Reject duplicate keys, missing required fields, unknown fields, wrong field types, and documents over 16 KiB. The test runner may mutate a copy of the fixture for negative cases. Keep the original staged manifest immutable.

| Field | Type | Pilot value |
| --- | --- | --- |
| `schema_revision` | Integer | `1` |
| `core_abi` | String | `0.1.0-draft` |
| `profile_revision` | Integer | `1` |
| `interface_id` | String | `org.ulabi.pilot.byte-copy` |
| `interface_version` | String | `1.0` |
| `platform_mapping` | String | `linux-x86_64-c` |
| `operation` | String | `copy_bytes` |
| `signature` | String | `(ptr<const u8>,u64,ptr<u8>,u64,ptr<u64>)->u32` |
| `export` | String | `ulabi_pilot_byte_copy_v1` |

These values are pilot fixtures. The signature encodes argument order, widths, and pointer direction. The reference design owns lifetime, validation order, and result semantics.

In each consumer, validate the manifest structure and exact fixture values before opening the library. A structurally valid manifest with a different platform mapping yields `unsupported_mapping` after all other fields match. Other manifest failures yield `manifest_rejected`. Open the supplied absolute library path and resolve the export from that library's handle. A failed open yields `load_error`. A successful open without the export yields `missing_export`. Release the handle after the binding and any call using it have finished.

A missing-export fixture is a valid shared library built by the requested provider toolchain. It exports only a separate marker and carries the unmodified manifest. This isolates symbol resolution from parsing and library loading.

## Case file format

The case file is a UTF-8 JSON object with integer `schema_revision` and ordered array `cases`. Each entry has a unique string `id`, a `kind` of `operation` or `binding`, one `input` object, and one `expected` object. Reject an unsupported revision or malformed entry before the matrix starts.

- For an operation, `input` contains `input_kind` (`small`, `generated`, or `null`), `input_length`, `output_kind` (`buffer` or `null`), `output_capacity`, and `written_pointer` (boolean). Its `expected` contains `status`, `written_length`, and `output` (`copied`, `unchanged`, or `absent`). The runner supplies the baseline manifest and normal provider library.
- For a binding case, `input` contains `manifest_edit` (null or an object with `kind` and, where needed, `field` and `value`) and `provider_role` (`normal`, `missing_library`, or `missing_export`). Manifest edit kinds are `truncate`, `duplicate`, `remove`, `set`, and `pad`. A duplicate repeats the existing value. For `pad`, `value` is the final byte length. Its `expected` contains `binding_outcome` and `load_attempts`. Derive the remaining expected fields using the binding-case rules below.

Pass the case file path to each consumer as `--cases <absolute-path>`. Consumers use the matching entry's `kind` and `input` to set up the call. The runner alone uses `expected` to decide whether the observation passes. For binding cases the runner stages the mutated manifest or requested provider artifact and the consumer runs its normal binding path.

## Harness contract

Implement `python3 reference/native-pilot/tests/run.py --out <directory>` from the repository root. A normal run builds both providers and both consumers into a new run directory under the given path. Implement `--artifacts <index.json>` to exercise independently prebuilt artifacts using the same checks. The artifact index records each language and role, library or executable path, paired manifest path for provider roles, SHA-256, and compiler version. Resolve relative paths against the index directory. Reject missing roles, repeated roles, identical C and Rust artifacts for the same role, or a digest mismatch before running cases.

The build uses the pinned local tools and local inputs. CI runs in a network-disabled environment with toolchains provisioned in advance. The Rust build uses offline mode with dependency versions locked if it uses Cargo dependencies. Build the C shared library and the Rust dynamic library as separate artifacts. Preserve tool versions and build logs. Use a clean environment for child processes, with inherited loader search overrides removed. Create the missing-library path under the new run directory and verify it is absent before launch.

Run cases in fixture order for these pairs, using a fresh process per case.

| Pair ID | Consumer | Provider |
| --- | --- | --- |
| `c-c` | C | C |
| `c-rust` | C | Rust |
| `rust-c` | Rust | C |
| `rust-rust` | Rust | Rust |

Invoke the consumer with `--library <absolute-path> --manifest <absolute-path> --case <id>`. The consumer’s test entry point uses the specified case to construct native arguments and then calls the same binding and invocation functions used by the normal consumer. The runner computes expectations independently. Each consumer writes one JSON object to stdout, using stderr for diagnostics. A tested failure such as a rejected manifest is an observed outcome and still exits zero when the consumer reports it successfully.

| Observation | Required representation |
| --- | --- |
| Binding outcome | `bound`, `manifest_rejected`, `unsupported_mapping`, `load_error`, or `missing_export` |
| Loader and call evidence | Integer `load_attempts` and `call_attempts`, incremented at the actual loader call and native call boundaries. All manifest failures require both counters to be zero. |
| Artifact selected | `loaded_library` is the canonical path returned by the binding layer for a successfully opened handle, or null. The runner checks it against the requested provider path and records its verified digest. |
| Native result | `status` and `written_length` as integers where observed, or null when no call occurred or no written-length address was supplied. |
| Output evidence | `output_length` and `output_sha256` for the whole output allocation, including guards, or null when there is no allocation. Calculate this after the call. |

Use a 10-second deadline per case and terminate the child process group on timeout. A crash, malformed result, nonzero exit, or timeout fails that case. If a build or artifact preflight fails, report the blocked matrix explicitly. Run all other independent cases after a case failure. A failure never causes substitution of a different provider.

## Case fixtures

Use MiB = 1,048,576 bytes. For operation cases, initialize an available written-length slot to `0xffffffffffffffff`. Fill the output allocation with `0xa5` and place 16 guard bytes of that value on either side of the declared output span. The small input vector is `00 01 7f 80 ff` (five bytes). Use byte `i mod 256` at offset `i` for generated payloads. Use separate valid storage for non-null arguments, including oversized extents in validation tests. A null entry in the case means an actual null address at the call boundary.

For a successful copy, the expected output allocation is the original guard, copied prefix, untouched tail, and final guard. For failures the allocation remains identical to its initialization. The harness independently constructs these bytes and compares hashes and lengths. Where capacity is zero and the output address is null, there is no output allocation. In the table, “unchanged” applies to any output storage that exists.

### Native operation cases

Every case in this table expects `bound`, one load attempt and one call attempt. Unless a row specifies otherwise, pointers are non-null, the input is the five-byte vector, and output capacity is 5.

| Case ID | Arguments changed | Status | Written length | Output |
| --- | --- | --- | --- | --- |
| `copy-empty-null` | Null input and output, both extents 0 | 0 | 0 | No allocation |
| `copy-empty-buffer` | Non-null input with length 0, output capacity 5 | 0 | 0 | Unchanged |
| `copy-exact` | Defaults | 0 | 5 | Exact five bytes |
| `copy-tail` | Output capacity 9 | 0 | 5 | Five bytes, tail unchanged |
| `copy-max` | Generated input length and output capacity both 1 MiB | 0 | 1 MiB | Exact generated payload |
| `short-buffer` | Output capacity 4 | 3 | 5 | Unchanged |
| `short-zero` | Null output, capacity 0 | 3 | 5 | No allocation |
| `null-input` | Null input, length 5 | 1 | 0 | Unchanged |
| `null-output` | Null output, capacity 5 | 1 | 0 | No allocation |
| `null-written` | Null written-length address | 1 | null | Unchanged |
| `input-limit` | Generated input length 1 MiB + 1 | 2 | 0 | Unchanged |
| `output-limit` | Output capacity 1 MiB + 1 | 2 | 0 | Unchanged |
| `written-before-limit` | Null written-length address and generated input length 1 MiB + 1 | 1 | null | Unchanged |
| `limit-before-data-null` | Null input with length 1 MiB + 1 | 2 | 0 | Unchanged |
| `data-null-before-capacity` | Null input with length 5 and output capacity 4 | 1 | 0 | Unchanged |

### Binding cases

Apply one change per row to the baseline fixture or artifact. Each case expects zero call attempts and null native result and output fields. Check that the missing-export case opened the expected alternate provider path, while a rejected manifest or failed load yields a null loaded path.

| Case ID | Change | Binding outcome | Load attempts |
| --- | --- | --- | --- |
| `bad-json` | Stage a truncated JSON object | `manifest_rejected` | 0 |
| `duplicate-key` | Repeat `interface_id` with its original value | `manifest_rejected` | 0 |
| `missing-field` | Remove `operation` | `manifest_rejected` | 0 |
| `unknown-field` | Add an `extra` field | `manifest_rejected` | 0 |
| `oversize-manifest` | Pad valid JSON with trailing spaces to 16 KiB + 1 | `manifest_rejected` | 0 |
| `wrong-field-type` | Change schema revision to the string `"1"` | `manifest_rejected` | 0 |
| `wrong-schema` | Change schema revision to 2 | `manifest_rejected` | 0 |
| `wrong-core` | Set Core ABI to `99.0.0` | `manifest_rejected` | 0 |
| `wrong-profile` | Change profile revision to 2 | `manifest_rejected` | 0 |
| `wrong-interface` | Append `.other` to the interface ID | `manifest_rejected` | 0 |
| `wrong-version` | Change interface version to `2.0` | `manifest_rejected` | 0 |
| `wrong-mapping` | Set platform mapping to `linux-aarch64-c` | `unsupported_mapping` | 0 |
| `wrong-operation` | Change operation to `other` | `manifest_rejected` | 0 |
| `wrong-signature` | Replace the first `u64` length with `u32` | `manifest_rejected` | 0 |
| `wrong-export` | Append `_other` to the export name | `manifest_rejected` | 0 |
| `missing-library` | Supply a known-absent library path | `load_error` | 1 |
| `missing-export` | Supply the alternate shared library with the baseline manifest | `missing_export` | 1 |

## Reports and release gate

Write `report.json` and `summary.txt` in the run directory. The JSON report records its schema revision, suite revision, source commit, target, toolchain versions, selected artifact and manifest digests, and an ordered result list. Each result records pair, case ID, expected and observed values, outcome, and the failing field or execution error. The summary gives counts and lists failing pair and case IDs with the reason. Publish both reports with the exact fixture revision and build logs.

| Runner exit | Meaning |
| --- | --- |
| `0` | All 128 required results passed on this target, with both cross-language directions represented. |
| `1` | At least one required pairing or case failed during a started matrix. |
| `2` | Configuration, target, toolchain, build, or artifact preflight blocked the matrix. |

A skipped or missing required result cannot pass the gate. Run it twice from clean directories on the pinned toolchains and compare pair IDs, case IDs, and semantic observations in order. Paths, wall-clock timings, and compiler build IDs are run metadata and can differ. The release gate requires equivalent semantic results.

## Implementation sequence and acceptance

1. Encode the manifest and case fixtures. Validate that IDs are unique, every expected outcome is explicit, and each of the 32 cases belongs to exactly one of the two case families.
2. Implement both consumer test entry points and the matching loader and call observations. Keep the build and linkage independent for each language. Prove one positive and two binding paths end to end with the C provider before wiring the remaining matrix.
3. Implement artifact preflight, execution, and reporting. Test the runner with fake children that time out, crash, and return malformed JSON. Verify the runner detects a substituted artifact and reports an absent matrix case.
4. Run all four pairings using both freshly built and prebuilt artifacts. Record the tested toolchain versions and confirm identical semantic observations on a second clean offline run.

Add a platform by supplying its mapping fixture and build logic while reusing semantic cases that apply unchanged. Add a language by introducing separate producer and consumer artifact roles and explicitly adding its pairings. Preserve existing case IDs. If the expected contract for an existing case changes, version the fixture and keep the previous revision's cases and evidence addressable.
