I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI before designing this document. The master design already establishes ULABI as language-, runtime-, OS-, CPU-, vendor-, Zamani-, and Sankofa-independent, and explicitly places return values, errors, ownership, calling conventions, and compatibility inside the Core ABI boundary.

I also checked the repository's current ABI documentation structure. docs/abi/core-abi.md, data-types.md, memory-model.md, and register-model.md already exist, so Design Document 10 should integrate with those documents rather than redefine them.

Design Document 10 — docs/abi/return-values.md

Copy everything below into:

docs/abi/return-values.md

# ULABI Return Values Specification

**Document:** `docs/abi/return-values.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Specification Area:** Core ABI — Return Values  
**Version:** 0.1.0-draft  
**Status:** Design Specification / Pre-Implementation  
**Normative Language:** MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, MAY  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the ULABI return-value contract.

The purpose of this specification is to ensure that independently implemented
components can return values across a ULABI boundary without requiring the
caller and callee to share:

- a programming language;
- a compiler;
- a runtime;
- an object model;
- a memory-management system;
- an operating system;
- a processor architecture;
- an error-handling mechanism.

ULABI return values are semantic ABI contracts.

A language's native return-value mechanism MUST be adapted to the ULABI
return-value model at the interoperability boundary.

---

# 2. Architectural Position

The relationship is:

```text
ULABI-DESIGN.md
        |
        v
ULABI-SPEC.md
        |
        v
docs/abi/core-abi.md
        |
        +-----------------------------+
        |                             |
        v                             v
calling-convention.md          data-types.md
        |                             |
        +-------------+---------------+
                      |
                      v
               return-values.md
                      |
          +-----------+-----------+
          |                       |
          v                       v
   exception-model.md      memory-model.md
          |                       |
          +-----------+-----------+
                      |
                      v
             interoperability

This document defines return semantics.

It does NOT define:

a programming-language exception syntax;

a specific CPU register assignment;

a specific native stack layout;

a specific garbage collector;

a specific object model;

a specific transport protocol.


Those concerns belong to their respective specifications.


---

3. Fundamental Principle

The fundamental rule is:

> A ULABI caller and callee MUST have an identical semantic interpretation of every successful return value and every failure result exchanged under the applicable contract.



The internal representation MAY differ.

For example:

Language A
    |
    | native return representation
    v
ULABI Return Contract
    ^
    | native return representation
    |
Language B

The language implementations do not need to use the same native mechanism.


---

4. Independence Requirement

ULABI MUST remain independent of every particular programming language.

For example:

C
C++
Rust
Go
Python
Java
Swift
Kotlin
Fortran
Ada
Zamani
Sankofa

MAY implement ULABI return values.

None of these languages defines the ULABI return-value model.

A conformant implementation MUST be possible without depending on Zamani or Sankofa.


---

5. Scope

This specification defines:

1. successful return values;


2. unit returns;


3. primitive returns;


4. structured returns;


5. aggregate returns;


6. optional returns;


7. result returns;


8. error returns;


9. multiple logical return values;


10. ownership of returned values;


11. lifetime of returned values;


12. mutability of returned values;


13. borrowed returns;


14. owned returns;


15. shared returns;


16. opaque-handle returns;


17. buffer returns;


18. streaming returns;


19. asynchronous returns;


20. cancellation;


21. partial results;


22. invalid return values;


23. malformed return representations;


24. return-value validation;


25. compatibility;


26. security;


27. architecture mappings;


28. conformance requirements.




---

6. Return Model

A ULABI operation has the logical form:

Call
 |
 v
Callee
 |
 +---- Success ----> ReturnValue
 |
 +---- Failure ----> Error

A return is therefore not merely a CPU register value.

It is a semantic contract.

Conceptually:

ReturnEnvelope {
    status
    value
    error
    ownership
    lifetime
    effects
}

The exact binary representation is determined by the applicable ABI and architecture mapping.


---

7. Return Status

Every ULABI invocation MUST have an unambiguous completion state.

At minimum:

SUCCESS
FAILURE

Profiles MAY define additional states where required.

Possible extended states include:

CANCELLED
TIMEOUT
PARTIAL
PENDING
STREAMING
RETRYABLE_FAILURE

An implementation MUST NOT introduce an additional semantic state that the caller cannot understand.


---

8. Successful Return

A successful operation returns a value matching its declared return type.

Example:

function add(
    a: I64,
    b: I64
) -> I64

A successful call MUST return exactly one semantic I64 value.

The caller MUST NOT reinterpret the returned representation as another type.


---

9. Unit Return

ULABI defines the semantic concept:

Unit

Unit means:

> The operation completed successfully but produces no data value.



Example:

function shutdown() -> Unit

A successful Unit return MUST be distinguishable from a failure.

The representation MAY be optimized by the physical calling convention.

The semantic meaning MUST remain equivalent.


---

10. Primitive Return Values

Primitive return values MAY include:

Bool

I8
I16
I32
I64
I128

U8
U16
U32
U64
U128

F32
F64

Char
Byte

The caller and callee MUST agree on:

type;

width;

representation;

alignment;

validity;

conversion rules.


Architecture-specific register placement belongs to docs/abi/calling-convention.md and docs/abi/register-model.md.


---

11. Structured Return Values

A function MAY return:

Record
Struct
Enum
Variant
Option
Result
Array
List
Map
Handle
Buffer

The return representation MUST preserve the semantic type.

Example:

Person {
    name: String
    age: U32
}

The caller MUST interpret the returned object according to the declared ULABI type descriptor.


---

12. Aggregate Returns

An aggregate is a return value consisting of multiple logical components.

Examples:

Record
Tuple
Struct
Array
Variant

The ABI MUST define whether an aggregate is:

returned directly;

returned through registers;

returned through caller-provided storage;

returned through an opaque handle;

returned through another ABI mechanism.


The logical semantics MUST remain independent of the physical mechanism.


---

13. Return Type Identity

Every externally visible return type MUST have a stable semantic identity.

Conceptually:

TypeID {
    namespace
    name
    version
}

The return type identity MUST NOT depend solely on:

native compiler type names;

source-language syntax;

compiler-generated names;

memory addresses;

native ABI-specific type identifiers.



---

14. Return Signature

A ULABI function signature includes:

FunctionID
Parameters
ReturnType
ErrorType
CallingConvention
OwnershipContract
LifetimeContract
EffectContract
CapabilityContract

Therefore:

Function
    |
    +-- Arguments
    |
    +-- Return Value
    |
    +-- Error
    |
    +-- Ownership
    |
    +-- Lifetime
    |
    +-- Effects

A change to the return type that changes binary or semantic interpretation MUST be treated as an ABI compatibility change.


---

15. Option Return

ULABI supports:

Option<T>

with:

None
Some(T)

Example:

function find_user(
    id: U64
) -> Option<User>

The caller MUST be able to distinguish:

None

from:

Some(User)

without relying on implementation-specific sentinel values.


---

16. Result Return

ULABI supports:

Result<T,E>

with:

Ok(T)
Err(E)

Example:

function divide(
    a: I64,
    b: I64
) -> Result<I64, DivisionError>

A successful result is:

Ok(value)

A failed result is:

Err(error)

The semantic distinction MUST be explicit.


---

17. Result vs ABI Failure

An important distinction exists between:

Result<T,E>

and a failure of the ABI invocation itself.

For example:

Result<I64, DivisionError>

may represent an application-level failure.

A malformed ABI message, invalid calling convention, or incompatible interface is a boundary-level failure.

Conceptually:

Invocation
 |
 +-- ABI Failure
 |
 +-- Valid Invocation
        |
        +-- Ok(T)
        |
        +-- Err(E)

This distinction MUST be preserved.


---

18. Error Return

An operation MAY declare an explicit error type.

Example:

function open_file(
    path: String
) -> Result<Handle<File>, FileError>

The error type MUST be part of the interface contract.

An implementation MUST NOT silently convert an error into a successful return.


---

19. Error Identity

Errors SHOULD have stable identities.

Conceptually:

ErrorID {
    namespace
    name
    version
    code
}

An error MAY contain structured information.

Example:

FileError {
    code: U32
    message: String
    path: String
}

Error representations MUST remain compatible with the applicable error model.


---

20. Native Exceptions

A language MAY use exceptions internally.

However:

> Native exceptions MUST NOT automatically cross a ULABI boundary as an unspecified implementation mechanism.



A language adapter MUST translate native exceptions into the declared ULABI error contract.

Example:

Native Exception
       |
       v
ULABI Adapter
       |
       v
Result<T,E>

This prevents one language's runtime exception model from becoming a requirement for every other language.


---

21. Panic / Fatal Failure

A native panic, abort, trap, or fatal runtime condition MUST NOT be represented as a normal successful return.

Where the applicable profile permits recovery, it SHOULD be translated into an explicit boundary failure.

If recovery is impossible, the implementation MUST follow the applicable process, runtime, or reliability profile.


---

22. Ownership of Returned Values

Every returned value crossing a memory boundary MUST have an ownership contract when ownership is relevant.

Possible states include:

OWNED
BORROWED
SHARED
IMMUTABLE_SHARED
TRANSFERRED
OPAQUE

Example:

function create_buffer() -> Owned<Buffer>

The caller becomes responsible for the returned resource according to the ownership contract.


---

23. Owned Return

An owned return transfers responsibility to the receiver.

Conceptually:

Callee
  |
  | ownership transfer
  v
Caller

After successful transfer, the caller MUST obey the applicable destruction or release contract.

The callee MUST NOT continue to mutate or release an owned object unless the contract explicitly permits shared ownership.


---

24. Borrowed Return

A borrowed return provides temporary access without transferring ownership.

Example:

function name(
    user: Borrowed<User>
) -> Borrowed<String>

The returned value MUST NOT outlive the lifetime defined by the contract.

The implementation MUST prevent or otherwise safely handle lifetime violations.


---

25. Shared Return

A shared return permits multiple parties to access a value.

The applicable profile MUST define:

sharing semantics;

mutation rules;

synchronization;

lifetime;

release;

ownership;

thread safety.


Shared mutable values MUST NOT be assumed safe without an explicit contract.


---

26. Immutable Return

An implementation MAY return an immutable value.

Example:

Immutable<String>

An immutable return MUST NOT be mutated through the returned ULABI reference.

This MAY enable:

zero-copy;

safe sharing;

caching;

concurrent reads.



---

27. Lifetime Contract

A returned reference, buffer, handle, or resource MUST have a defined lifetime.

Possible lifetime models include:

CALL
SCOPE
BORROW
OWNED
SHARED
UNTIL_RELEASE
UNTIL_CLOSE
SESSION
PROCESS
EXPLICIT

A receiver MUST NOT assume a longer lifetime than the contract guarantees.


---

28. Caller-Allocated Return Storage

A physical ABI MAY use caller-provided storage for large return values.

Conceptually:

Caller
  |
  | provides return storage
  v
Callee
  |
  | writes return value
  v
Caller Storage

This is a physical optimization.

It MUST preserve the same logical ownership and lifetime semantics as a direct return.


---

29. Indirect Return

Large aggregates MAY be returned indirectly.

The ABI mapping MUST define:

storage location;

alignment;

size;

initialization;

ownership;

failure behavior;

cleanup behavior.


The logical return type remains unchanged.


---

30. Handle Returns

Opaque resources SHOULD be returned using handles where direct memory sharing is unsafe or impossible.

Examples:

Handle<File>
Handle<Device>
Handle<Process>
Handle<Stream>
Handle<GPUBuffer>

A handle MUST NOT expose implementation-specific internal addresses.


---

31. Buffer Returns

A returned buffer MUST define:

data reference
length
capacity
element size
alignment
mutability
ownership
lifetime

Example:

Buffer<U8>

The buffer contract MUST define whether the caller owns the buffer or merely borrows it.


---

32. Zero-Length Returns

A zero-length collection is a valid value.

Examples:

Bytes(length=0)
List<T>(length=0)
String(length=0)

Zero length MUST NOT automatically mean:

null
invalid
error
uninitialized

unless the type contract explicitly defines such semantics.


---

33. Null

ULABI MUST NOT use an implicit universal null convention.

Absence SHOULD be represented using:

Option<T>

or another explicitly defined semantic type.

A native null pointer MAY be adapted into:

None

where the interface contract permits it.


---

34. Multiple Return Values

A function MAY logically return multiple values.

Example:

function divide(
    a: I64,
    b: I64
) -> (I64, I64)

The preferred universal representation is a structured aggregate such as:

Tuple<I64, I64>

or:

Record {
    quotient: I64
    remainder: I64
}

The physical ABI MAY use multiple registers where supported.


---

35. Return Value Ordering

For multiple logical return values, ordering MUST be deterministic.

Example:

(a, b, c)

MUST always map to:

return[0] = a
return[1] = b
return[2] = c

unless named fields explicitly define identity.


---

36. Streaming Returns

A function MAY return a stream.

Example:

Stream<Bytes>

A stream is not equivalent to a single collection.

The streaming profile MUST define:

item boundaries;

completion;

failure;

cancellation;

backpressure;

ownership;

lifetime.



---

37. Asynchronous Returns

An asynchronous operation MAY return:

Future<T>

Example:

function fetch_data() -> Future<Bytes>

The Future<T> represents eventual completion.

It MUST NOT be confused with:

T

A synchronous caller MUST NOT assume that an asynchronous return is already complete.


---

38. Pending State

An asynchronous operation MAY have:

PENDING

state.

The applicable async profile MUST define:

completion;

polling;

waiting;

cancellation;

timeout;

ownership;

failure;

resource cleanup.



---

39. Cancellation

A cancellable operation MUST define whether cancellation:

prevents execution;

interrupts execution;

requests cooperative cancellation;

completes with a cancellation error;

leaves partial work;

releases resources.


Cancellation MUST NOT silently produce a successful value unless the contract explicitly defines cancellation as successful completion.


---

40. Partial Results

Some operations MAY return partial results.

Example:

Result<Partial<Data>, Error>

or an explicit:

PARTIAL

completion state.

The interface MUST define:

whether partial data is usable;

whether it is owned;

whether it can be resumed;

whether the operation can be retried;

whether the partial result is deterministic.



---

41. Retryable Returns

A failure MAY be explicitly classified as retryable.

Example:

Error {
    code: TIMEOUT
    retryable: true
}

Retry semantics MUST NOT be inferred merely from an error name.

The contract SHOULD define:

retry safety;

idempotency;

retry limits;

backoff;

state preservation.



---

42. Idempotency

A returned failure MUST NOT cause a caller to retry blindly.

If the operation changes external state, the caller MUST know whether retrying is safe.

Function contracts SHOULD declare:

Idempotent
NonIdempotent
ConditionallyIdempotent


---

43. Determinism

For deterministic functions:

same input
+
same defined environment
=
same semantic return value

Where nondeterminism exists, the interface SHOULD declare it.

Examples:

Randomness
Time
ExternalDevice
Network
NonDeterministic


---

44. Return Value Validation

The receiver MUST validate externally supplied return representations according to the applicable contract.

Validation SHOULD include:

type identity;

version;

length;

bounds;

alignment;

discriminant;

ownership metadata;

lifetime metadata;

capability requirements;

encoding;

canonical representation.


Invalid values MUST NOT be interpreted as valid values.


---

45. Malformed Return

A malformed return is different from an application-level error.

Example:

Application:
    Err(FileNotFound)

ABI:
    malformed return representation

The first is a valid semantic return.

The second is a boundary integrity failure.


---

46. Type Confusion

An implementation MUST reject or safely isolate a return value whose declared type does not match the expected type.

Example:

Expected:
    I64

Received:
    String

This MUST NOT be silently converted unless an explicit conversion contract exists.


---

47. Integer Conversion

A return value MUST NOT be silently narrowed.

For example:

I64 -> I32

requires explicit conversion semantics.

Possible outcomes include:

success
overflow
conversion error
saturation

The interface contract MUST determine the behavior.


---

48. Floating-Point Conversion

Floating-point return conversions MUST explicitly define behavior for:

NaN;

infinity;

signed zero;

overflow;

underflow;

precision loss.


A receiver MUST NOT assume that all native floating-point formats are interchangeable.


---

49. String Returns

A returned String MUST expose sufficient information to determine:

encoding
length
data

ULABI SHOULD use UTF-8 as the canonical representation.

Null termination MUST NOT be required.


---

50. Byte Returns

Binary data MUST remain distinct from text.

Example:

Bytes

MUST NOT be interpreted as:

String

without an explicit conversion.


---

51. Security Requirements

Return values are untrusted boundary data unless the security profile explicitly establishes trust.

Implementations MUST defend against:

buffer overflow;

integer overflow;

malformed lengths;

invalid discriminants;

type confusion;

use-after-release;

stale handles;

capability escalation;

malicious resource sizes;

resource exhaustion;

unauthorized memory access;

forged opaque references.



---

52. Resource Limits

Implementations SHOULD enforce configurable limits for:

maximum return size;

maximum nesting depth;

maximum collection length;

maximum string size;

maximum buffer size;

maximum number of returned handles;

maximum stream item size.


A resource-limit failure MUST be distinguishable from a normal application error where required by the applicable profile.


---

53. Capability Security

A returned capability MUST carry only the authority explicitly granted by the contract.

A returned handle MUST NOT automatically grant additional capabilities.

For example:

Handle<FileRead>

MUST NOT implicitly grant:

FileWrite
Delete
Execute

unless those capabilities were explicitly authorized.


---

54. Cross-Process Returns

For out-of-process operation, raw native pointers MUST NOT normally be returned.

The implementation SHOULD use:

Handle
Capability
SerializedValue
SharedMemoryDescriptor

according to the applicable profile.


---

55. Cross-Machine Returns

For distributed operation, return values MUST be transport-independent.

The return contract MUST survive differences in:

CPU architecture;

byte order;

pointer width;

operating system;

language;

runtime.


Portable representations MUST therefore use explicitly defined encodings.


---

56. Return Value and Calling Convention

The logical return model is:

ULABI Logical Return
        |
        +-- Status
        +-- Value
        +-- Error
        +-- Ownership
        +-- Lifetime
        |
        v
Architecture Mapping
        |
        +-- Registers
        +-- Stack
        +-- Caller Storage
        +-- ABI-specific mechanism

The physical calling convention MUST preserve the logical meaning.


---

57. Register Returns

A physical ABI MAY return values in registers.

The register assignment MUST be defined by the architecture-specific mapping.

This document intentionally does not prescribe:

x86-64 registers;

ARM registers;

RISC-V registers;

GPU registers;

NPU registers;

future architecture registers.


The logical return contract remains architecture-neutral.


---

58. Stack Returns

A physical ABI MAY use stack-based return mechanisms.

The stack layout MUST be defined by the applicable calling convention.

The logical return semantics MUST remain unchanged.


---

59. Caller-Provided Return Buffers

Large return values MAY use caller-provided storage.

The implementation MUST define:

who allocates
who initializes
who writes
who owns
who releases
how failure is handled

These rules MUST be deterministic.


---

60. Return Destruction

If a return value transfers ownership, the receiving side becomes responsible for destruction or release according to the applicable memory/resource contract.

The producer MUST NOT destroy the object after ownership transfer unless shared ownership is explicitly specified.


---

61. Return Aliasing

The specification MUST define whether returned values may alias:

input parameters;

global state;

other return values;

shared memory;

internal runtime objects.


Aliasing MUST NOT be assumed safe unless explicitly guaranteed.


---

62. Mutable Returns

If a returned value is mutable, the contract MUST define:

who may mutate it;

whether mutation is visible to the producer;

whether synchronization is required;

whether aliasing is permitted;

when mutation becomes invalid.



---

63. Immutable Returns and Optimization

Immutable returns MAY be optimized using:

sharing;

interning;

caching;

zero-copy;

reference reuse.


Such optimization MUST preserve observable ULABI semantics.


---

64. Zero-Copy Returns

Zero-copy return paths MAY be used when:

ownership is explicit;

lifetime is explicit;

memory remains valid;

alignment is guaranteed;

access rights are enforced;

aliasing rules are satisfied.


Zero-copy MUST NOT weaken memory safety.


---

65. Serialization Boundary

When a return crosses a serialization boundary, the serialized representation MUST preserve the semantic return contract.

Serialization MUST preserve:

type identity;

value;

error state;

variant state;

ownership semantics where representable;

required metadata.


Native pointers MUST NOT be serialized as portable references.


---

66. Compatibility

A compatible ABI version MUST preserve the interpretation of existing return values.

The following changes generally require a new incompatible contract:

changing a return type;

changing a return discriminant;

changing ownership semantics;

changing lifetime semantics;

changing representation in a way that changes interpretation;

removing a previously valid return variant;

changing error semantics incompatibly.



---

67. Adding Optional Return Information

A compatible extension MAY add optional metadata if existing implementations can safely ignore it.

For example:

ReturnValue {
    value
    optional_metadata
}

Unknown optional metadata MUST NOT alter the interpretation of the known value.


---

68. Unknown Variants

If a return type permits future variants, the contract MUST define what an older implementation does when it receives an unknown variant.

Possible behavior:

preserve
ignore
return UnsupportedVariant
reject
escalate

The behavior MUST be specified.


---

69. Version Negotiation

Before using a return contract that requires an optional feature, implementations SHOULD negotiate the applicable profile or capability.

Example:

Caller
  |
  | capability negotiation
  v
Callee
  |
  | agreed return contract
  v
Call

An implementation MUST NOT assume support for optional return mechanisms that were not negotiated or declared.


---

70. Backward Compatibility

A newer implementation SHOULD be able to consume return values produced by older compatible implementations.

Backward compatibility MUST be based on semantic and binary compatibility, not merely matching version strings.


---

71. Forward Compatibility

Where forward compatibility is supported, older implementations MUST safely handle newer return metadata or optional extensions.

They MUST NOT reinterpret unknown data as a known type.


---

72. FFI Requirements

A language adapter implementing ULABI MUST translate the language's native return model into the ULABI contract.

Examples:

C
    native return
        |
        v
    ULABI adapter

Rust
    Result<T,E>
        |
        v
    ULABI Result<T,E>

Python
    object / exception
        |
        v
    ULABI value / error

Java
    object / exception
        |
        v
    ULABI value / error

The native mechanism is implementation-specific.


---

73. No Native ABI Leakage

A ULABI implementation MUST NOT expose native ABI assumptions as universal requirements.

Examples of prohibited implicit assumptions:

pointer == universal reference
int == I32
long == I64
null == Option::None
native exception == ULABI error
native object layout == ULABI record
native struct padding == ULABI layout

Adapters MUST make such mappings explicit.


---

74. Compiler Requirements

A compiler targeting ULABI SHOULD be capable of generating:

return-value metadata;

return type descriptors;

ownership metadata;

lifetime metadata;

error metadata;

calling-convention information;

compatibility information.


The compiler MUST preserve the declared ULABI contract.


---

75. Runtime Requirements

A runtime implementing ULABI SHOULD provide mechanisms for:

return-value validation;

ownership enforcement;

lifetime enforcement;

resource release;

error translation;

asynchronous completion;

cancellation;

capability checking.


A runtime MUST NOT silently alter the declared return semantics.


---

76. Validator Requirements

A ULABI validator SHOULD verify:

Return Type
Return Representation
Error Contract
Ownership
Lifetime
Alignment
Size
Discriminants
Capabilities
Version
Compatibility

Invalid contracts MUST be rejected before unsafe execution where practical.


---

77. Debugging Requirements

Debug metadata SHOULD allow tools to determine:

function
return type
return value location
return status
error
ownership
lifetime

Debugging information MUST NOT change the actual ABI semantics.


---

78. Observability

Where the observability profile is enabled, return events MAY expose:

function ID
interface ID
status
duration
error identity
result size
resource usage

Sensitive returned data MUST NOT automatically be logged.


---

79. Privacy

Implementations MUST NOT assume that return values are safe to expose to:

logs;

telemetry;

traces;

debuggers;

monitoring systems.


Sensitive data SHOULD be explicitly classified.


---

80. Fault Isolation

A malformed or invalid return MUST NOT corrupt unrelated execution state.

Where possible:

Invalid Return
      |
      v
Validation Failure
      |
      v
Boundary Containment
      |
      v
Caller Continues / Fails Safely


---

81. Recovery

Recovery from a return-value failure MUST follow the applicable reliability policy.

The implementation MUST NOT automatically retry an operation merely because return validation failed.

Recovery policy MUST distinguish:

Application Failure
ABI Failure
Transport Failure
Memory Failure
Security Failure
Resource Exhaustion


---

82. Self-Healing Interaction

The ULABI self-healing profile MAY use return-value failures as evidence.

The sequence is:

Return Failure
     |
     v
Collect Evidence
     |
     v
Classify Failure
     |
     +---- Known Safe Recovery
     |          |
     |          v
     |       Recover
     |          |
     |          v
     |       Verify
     |
     +---- Unknown / Unsafe
                |
                v
             Escalate

Self-healing MUST NOT modify ABI semantics autonomously.


---

83. Formal Invariants

The following invariants are REQUIRED.

RV-001

A successful return MUST conform to its declared return type.

RV-002

A failure MUST NOT be represented as a successful value.

RV-003

A return value MUST NOT be interpreted using an incompatible type.

RV-004

Ownership MUST be explicit where ownership matters.

RV-005

Lifetime MUST be explicit for returned references and resources.

RV-006

Unknown optional metadata MUST NOT change known value semantics.

RV-007

Unknown variants MUST follow the defined compatibility rule.

RV-008

Malformed return representations MUST NOT be interpreted as valid values.

RV-009

Native language exception mechanisms MUST NOT silently define the ULABI error model.

RV-010

Architecture-specific return mechanisms MUST preserve the logical return contract.

RV-011

A returned capability MUST NOT grant undeclared authority.

RV-012

Zero-copy optimization MUST NOT violate ownership, lifetime, aliasing, or memory-safety rules.

RV-013

A return value MUST be deterministic where the interface declares deterministic behavior.

RV-014

A caller MUST NOT assume support for an undeclared optional return feature.


---

84. Failure Modes

Implementations MUST account for at least:

InvalidType
InvalidLength
InvalidEncoding
InvalidDiscriminant
InvalidOwnership
InvalidLifetime
InvalidCapability
ReturnBufferTooSmall
ReturnBufferMisaligned
UnsupportedReturnType
UnsupportedVariant
UnsupportedFeature
ResourceLimitExceeded
MalformedReturn
ABIIncompatible
ABIValidationFailure
Cancelled
Timeout
RemoteFailure
TransportFailure

The exact error-code registry is defined elsewhere.


---

85. Conformance Levels

A return-value implementation MAY declare:

ULABI-RET-BASIC
ULABI-RET-STRUCTURED
ULABI-RET-ERROR
ULABI-RET-OWNERSHIP
ULABI-RET-ASYNC
ULABI-RET-STREAMING
ULABI-RET-ZEROCOPY
ULABI-RET-DISTRIBUTED
ULABI-RET-SECURE

A complete implementation MUST publish exactly which capabilities it supports.


---

86. Conformance Tests

The conformance suite MUST eventually test at least:

Primitive returns

Bool

I8

I16

I32

I64

I128

U8

U16

U32

U64

U128

F32

F64

Char

Byte


Structural returns

Record

Tuple

Enum

Variant

Array

List


Semantic returns

Unit

Option

Result

Handle

Buffer


Ownership

owned return;

borrowed return;

shared return;

immutable return;

transfer;

release.


Failure

application error;

malformed return;

invalid type;

invalid length;

unsupported variant;

ABI mismatch.


Compatibility

old caller/new callee;

new caller/old callee;

optional metadata;

unknown variants;

negotiated features.


Security

forged handle;

oversized return;

invalid buffer;

invalid capability;

type confusion;

use-after-release.



---

87. Reference Test Example

Conceptual test:

function echo(value: String) -> String

Test:

Input:
    "ULABI"

Expected:
    "ULABI"

The test MUST verify semantic equality, not merely native memory equality.


---

88. Cross-Language Test

At minimum, the eventual reference suite SHOULD test:

Implementation A
       |
       v
    ULABI
       |
       v
Implementation B

where A and B use different implementation languages.

The test MUST verify:

same input
same contract
same semantic return


---

89. Architecture Matrix

The conformance suite SHOULD eventually test return-value mappings on:

x86-64
AArch64
RISC-V
WASM
embedded targets
future architectures

The logical return contract MUST remain unchanged.


---

90. Reference Implementation Boundary

The reference implementation MUST NOT become the definition of the standard.

The hierarchy is:

Specification
     |
     v
Reference Tests
     |
     +---- Reference Implementation A
     |
     +---- Reference Implementation B
     |
     +---- Independent Implementation C

The specification remains authoritative.


---

91. Required Integration

This document integrates with:

ULABI-DESIGN.md
ULABI-SPEC.md

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md

docs/runtime/runtime-interface.md
docs/runtime/async-model.md
docs/runtime/resource-management.md

docs/memory/memory-safety.md
docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/shared-memory.md

docs/security/security-model.md
docs/security/capability-security.md

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md

docs/standards/conformance.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md

These documents MUST NOT redefine the return-value rules established here.

They MUST reference this document for return semantics.


---

92. Integration Contract for Future Documents

Once this document is accepted:

Calling convention

MUST define only the physical mapping of logical return values.

Register model

MUST define where eligible return values are placed in registers.

Stack model

MUST define stack-based return mechanisms.

Exception model

MUST define how boundary failures and native exception translation interact.

Memory model

MUST define ownership, lifetime, aliasing, and memory validity.

FFI

MUST define language-specific adaptation into the ULABI return contract.

Async model

MUST define Future and asynchronous completion semantics.

Distributed ABI

MUST define serialized return semantics.

Security model

MUST define validation, authority, and trust requirements.

Conformance

MUST define executable tests for the requirements in this document.


---

93. Required Schemas

The following machine-readable schemas SHOULD eventually exist:

schemas/return-value.schema.json
schemas/return-type.schema.json
schemas/result.schema.json
schemas/option.schema.json
schemas/error.schema.json
schemas/ownership.schema.json
schemas/lifetime.schema.json
schemas/buffer-return.schema.json
schemas/async-return.schema.json

Schemas MUST be versioned.


---

94. Required Examples

The repository SHOULD eventually contain:

examples/returns/
├── primitive-return/
├── unit-return/
├── struct-return/
├── option-return/
├── result-return/
├── error-return/
├── owned-return/
├── borrowed-return/
├── shared-return/
├── handle-return/
├── buffer-return/
├── async-return/
├── streaming-return/
├── distributed-return/
└── compatibility/


---

95. Required Test Files

The repository SHOULD eventually contain:

tests/abi/returns/
├── primitive_returns.*
├── structured_returns.*
├── option_returns.*
├── result_returns.*
├── error_returns.*
├── ownership_returns.*
├── lifetime_returns.*
├── handle_returns.*
├── buffer_returns.*
├── async_returns.*
├── streaming_returns.*
├── malformed_returns.*
├── compatibility_returns.*
├── security_returns.*
└── cross_language_returns.*

The extension MUST be selected according to the implementation language.

The test semantics MUST remain language-neutral.


---

96. Required Conformance Modules

The conformance system SHOULD eventually contain:

conformance/return_values/
├── primitive/
├── structured/
├── option/
├── result/
├── errors/
├── ownership/
├── lifetime/
├── handles/
├── buffers/
├── async/
├── streaming/
├── compatibility/
├── malformed/
└── security/

Each test MUST identify the ULABI requirement it verifies.


---

97. Required Reference Implementation Modules

A language-neutral reference implementation architecture SHOULD eventually contain:

reference/
├── abi/
│   ├── return_value/
│   ├── result/
│   ├── option/
│   ├── error/
│   ├── ownership/
│   ├── lifetime/
│   ├── handle/
│   └── buffer/
├── validation/
└── compatibility/

The exact implementation language is intentionally unspecified.


---

98. Required Implementation Modules

An implementation SHOULD eventually separate the return-value subsystem into:

implementations/
<implementation>/
├── abi/
│   ├── return_values
│   ├── return_status
│   ├── return_types
│   ├── return_validation
│   ├── return_encoding
│   ├── return_decoding
│   ├── return_ownership
│   ├── return_lifetime
│   ├── return_errors
│   ├── return_handles
│   ├── return_buffers
│   ├── return_async
│   └── return_streaming

The implementation MUST NOT collapse all of these concerns into one architecture-specific module if doing so prevents independent validation.


---

99. Dependency Direction

The dependency direction MUST be:

semantic return contract
          |
          v
return type model
          |
          v
validation
          |
          v
encoding / decoding
          |
          v
logical calling convention
          |
          v
architecture mapping
          |
          v
native implementation

The reverse direction MUST NOT define the semantic contract.

For example:

x86 register convention

MUST NOT determine what a ULABI Result<T,E> means.


---

100. Required Module Interfaces

The implementation SHOULD expose conceptual interfaces equivalent to:

ReturnValue
ReturnStatus
ReturnType
ReturnError
ReturnEnvelope
ReturnValidator
ReturnEncoder
ReturnDecoder
OwnershipContract
LifetimeContract
ReturnBuffer
ReturnHandle
AsyncReturn
StreamReturn

These names are conceptual API/module names, not mandatory programming-language syntax.


---

101. ReturnValue Interface

Conceptually:

ReturnValue {
    type_id
    value
    ownership
    lifetime
}

Required operations:

type()
validate()
ownership()
lifetime()


---

102. ReturnValidator Interface

Conceptually:

ReturnValidator.validate(
    expected_type,
    returned_value,
    contract
)

It MUST verify the applicable return invariants before the value is exposed to the caller.


---

103. ReturnEncoder Interface

Conceptually:

ReturnEncoder.encode(
    return_value,
    encoding_profile
)

The encoder MUST produce the representation defined by the applicable ABI profile.


---

104. ReturnDecoder Interface

Conceptually:

ReturnDecoder.decode(
    bytes,
    expected_type,
    encoding_profile
)

The decoder MUST validate the representation before producing a usable semantic return value.


---

105. ReturnError Interface

Conceptually:

ReturnError {
    error_id
    category
    code
    details
    retryable
}

The implementation MAY add metadata but MUST preserve the declared semantic error.


---

106. Ownership Interface

Conceptually:

OwnershipContract {
    mode
    transfer
    release
}

Ownership rules MUST be enforced consistently with the memory subsystem.


---

107. Lifetime Interface

Conceptually:

LifetimeContract {
    scope
    expires_when
    release_required
}

A returned resource MUST NOT remain usable after its contract expires.


---

108. Buffer Interface

Conceptually:

ReturnBuffer {
    data
    length
    capacity
    element_size
    alignment
    ownership
    lifetime
    mutability
}

The final binary layout belongs to the memory and encoding specifications.


---

109. Async Interface

Conceptually:

AsyncReturn<T> {
    state
    poll()
    wait()
    cancel()
}

The exact async API belongs to:

docs/runtime/async-model.md

This document defines only the return-value semantics.


---

110. Streaming Interface

Conceptually:

StreamReturn<T> {
    next()
    cancel()
    close()
}

The streaming profile MUST define the complete protocol.


---

111. Implementation Independence

No implementation MAY make the normative specification dependent upon:

C ABI;

C++ ABI;

Rust ABI;

JVM ABI;

Python C API;

.NET ABI;

WebAssembly ABI;

Zamani ABI;

Sankofa ABI.


These MAY be implementation mappings.

They are not the ULABI definition.


---

112. Completion Criteria

This document is considered complete for the current specification phase when:

return semantics are explicitly defined;

success and failure are distinguished;

Result and Option semantics are defined;

ownership is defined;

lifetime is defined;

aggregate returns are defined;

indirect returns are defined;

handle returns are defined;

buffer returns are defined;

async and streaming boundaries are defined;

malformed returns are defined;

compatibility rules are defined;

security requirements are defined;

conformance requirements are defined;

required schemas are identified;

required tests are identified;

required implementation modules are identified;

integration dependencies are explicitly declared.


Future implementation work MUST implement these contracts rather than silently changing them.


---

113. Normative Summary

A conformant ULABI return implementation:

1. MUST return values according to declared ULABI types.


2. MUST distinguish success from failure.


3. MUST preserve semantic type identity.


4. MUST preserve ownership semantics.


5. MUST preserve lifetime semantics.


6. MUST validate boundary values.


7. MUST reject incompatible or malformed representations.


8. MUST NOT leak native ABI assumptions into the universal contract.


9. MUST preserve logical semantics across architecture mappings.


10. MUST protect against type confusion and memory-safety failures.


11. MUST explicitly define asynchronous and streaming behavior when supported.


12. MUST respect compatibility rules.


13. MUST expose supported return capabilities for conformance.


14. MUST remain independent of every particular programming language, compiler, runtime, operating system, processor, vendor, Zamani, and Sankofa.




---

114. Final Architectural Rule

> ULABI defines what a return value means at an interoperability boundary. An implementation defines how that return value is physically represented.



Therefore:

Semantic Contract
       |
       v
ULABI Return Model
       |
       +-------------------+
       |                   |
       v                   v
Architecture Mapping   Language Adapter
       |                   |
       v                   v
Physical ABI          Native ABI

The physical implementation may change.

The ULABI semantic contract remains stable.

## Required implementation work — ordered so files do not need to be reopened unnecessarily

The important point is that **`return-values.md` should be treated as the contract boundary**. We should not start writing implementation code before the lower-level contracts it depends upon are fixed.

### Phase 1 — independent semantic foundation

These are the files/modules I would complete first:

| Order | File/module | Responsibility |
|---|---|---|
| 1 | `schemas/return-type.schema.json` | Machine-readable return-type contract |
| 2 | `schemas/return-value.schema.json` | Return envelope/status/value contract |
| 3 | `schemas/result.schema.json` | `Result<T,E>` |
| 4 | `schemas/option.schema.json` | `Option<T>` |
| 5 | `schemas/error.schema.json` | Error identity and representation |
| 6 | `schemas/ownership.schema.json` | Ownership metadata |
| 7 | `schemas/lifetime.schema.json` | Lifetime metadata |
| 8 | `docs/abi/return-values.md` | **The document above — normative return contract** |

These are intentionally independent of any programming language.

---

# Phase 2 — ABI dependencies

Then complete the ABI documents that physically integrate the return contract:

```text
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md

Their responsibilities must remain separate:

calling-convention.md

Defines:

logical return
        ↓
physical return
        ↓
register/stack/indirect mapping

It must not redefine what Result, Option, ownership, or lifetime mean.

register-model.md

Defines where eligible return values physically go.

stack-model.md

Defines stack-based return mechanisms.

memory-model.md

Defines the memory validity behind returned references/buffers.

exception-model.md

Defines translation between native exception mechanisms and ULABI failures.

data-types.md

Defines the actual binary/type representations.


---

Phase 3 — memory/resource modules

These are required because return values can transfer resources:

implementations/<impl>/memory/
├── ownership
├── lifetime
├── allocation
├── buffer
├── handle
├── reference
└── validation

Conceptual responsibilities:

ownership
    ↓
who owns returned object?

lifetime
    ↓
how long is it valid?

allocation
    ↓
who allocated it?

buffer
    ↓
how is returned memory described?

handle
    ↓
how are opaque resources represented?

reference
    ↓
how are borrowed values represented?

validation
    ↓
is returned memory legal?


---

Phase 4 — return-value implementation modules

Only after the contracts above are stable should the actual implementation modules be created.

The recommended module decomposition is:

implementations/<implementation>/abi/return/
│
├── mod.*
├── value.*
├── status.*
├── type.*
├── envelope.*
├── validator.*
├── encoder.*
├── decoder.*
├── error.*
├── option.*
├── result.*
├── aggregate.*
├── ownership.*
├── lifetime.*
├── handle.*
├── buffer.*
├── async.*
├── stream.*
├── compatibility.*
└── security.*

What each module does

mod.*

Public subsystem boundary and exports.

value.*

Generic semantic return value.

status.*

SUCCESS
FAILURE
CANCELLED
PENDING
PARTIAL
...

type.*

Return-type identity and descriptors.

envelope.*

Combines status/value/error/metadata.

validator.*

Validates return values before exposure.

encoder.*

Converts semantic return values into the selected ABI representation.

decoder.*

Converts ABI representations into validated semantic values.

error.*

ULABI error representation.

option.*

Option<T>.

result.*

Result<T,E>.

aggregate.*

Tuple/record/struct/array returns.

ownership.*

Return ownership transfer.

lifetime.*

Returned-value lifetime enforcement.

handle.*

Opaque resource returns.

buffer.*

Returned buffers.

async.*

Future<T> / asynchronous returns.

stream.*

Stream<T>.

compatibility.*

Version and feature compatibility.

security.*

Boundary validation and anti-confusion/resource-limit enforcement.


---

Phase 5 — FFI integration

Then create:

implementations/<implementation>/ffi/
├── return_adapter.*
├── exception_adapter.*
├── ownership_adapter.*
├── lifetime_adapter.*
└── type_adapter.*

The critical architecture is:

Native Language
      |
      v
Native ABI
      |
      v
FFI Adapter
      |
      v
ULABI Return Contract

Not:

ULABI = C ABI

and not:

ULABI = Rust ABI


---

Phase 6 — conformance implementation

Then:

conformance/return_values/
├── primitive/
├── structured/
├── option/
├── result/
├── errors/
├── ownership/
├── lifetime/
├── handles/
├── buffers/
├── async/
├── streaming/
├── malformed/
├── compatibility/
├── security/
└── cross_language/

Each test should reference an invariant such as:

RV-001
RV-002
RV-003
...
RV-014

That gives us traceability:

Specification
     ↓
Requirement ID
     ↓
Schema
     ↓
Implementation
     ↓
Conformance Test


---

Complete code/module names for the return-value subsystem

The final implementation should therefore have, at minimum:

return_value
return_status
return_type
return_envelope
return_validator
return_encoder
return_decoder
return_error
return_option
return_result
return_aggregate
return_ownership
return_lifetime
return_handle
return_buffer
return_async
return_stream
return_compatibility
return_security

And supporting foundational modules:

type_id
interface_id
function_id
error_id
capability
ownership_contract
lifetime_contract
buffer_descriptor
handle_descriptor
encoding_profile
abi_version
feature_set

Integration graph

The intended final dependency direction is:

ULABI-DESIGN.md
                           │
                           ▼
                      ULABI-SPEC.md
                           │
                           ▼
                     Core ABI
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
         Data Types   Calling Model   Memory Model
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Return Contract
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Validation       Ownership        Lifetime
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                  Encoding / Decoding
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           Local          IPC        Distributed
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                       FFI Layer
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Language A       Language B       Language C
                           │
                           ▼
                     Conformance

This is deliberately consistent with the existing Core ABI design, which already specifies that return-values.md is a specialized document beneath the Core ABI and that specialized documents must not contradict it.

One important rule for the next documents

Do not modify return-values.md later merely because another document is written. Instead, the subsequent documents must implement the integration contracts already declared here. If a genuine contradiction is discovered, that should be treated as a specification change/versioned design decision—not normal cleanup.

That gives us the workflow you asked for:

Finish Document
       ↓
Finish its schemas
       ↓
Finish its implementation modules
       ↓
Finish its conformance tests
       ↓
Integrate downstream
       ↓
Do NOT reopen the completed document

So docs/abi/return-values.md is now the return-value contract, while the next implementation work belongs in the schemas, ABI mapping, runtime, FFI, and conformance layers listed above.