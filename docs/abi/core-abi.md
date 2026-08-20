I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI. It is already substantially developed: it establishes the language-neutral architecture, Core + Profiles model, interoperability modes, semantic types, memory boundaries, security, self-healing, distributed operation, hardware, compatibility, and conformance direction.

So Design Document 3 should not repeat the master architecture. It should drill down into the actual ABI foundation:

> docs/abi/core-abi.md — ULABI Core ABI Specification



This is the document that should eventually answer: “Exactly what must two independently implemented systems agree upon at the binary boundary?”

Copy/paste this as docs/abi/core-abi.md:

# ULABI Core ABI

**Document:** `docs/abi/core-abi.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Specification Area:** Core ABI  
**Version:** 0.1.0-draft  
**Status:** Design Specification / Pre-Implementation  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the design of the ULABI Core ABI.

The Core ABI is the smallest stable binary interoperability layer required for
independently implemented software components to communicate.

The Core ABI defines the boundary contract between:

- programming languages;
- compilers;
- runtimes;
- libraries;
- applications;
- processes;
- operating systems;
- execution environments;
- hardware-specific implementations.

The Core ABI MUST remain independent of any particular:

- programming language;
- compiler;
- runtime;
- operating system;
- processor architecture;
- memory-management strategy;
- vendor;
- project.

The Core ABI is therefore the foundation upon which ULABI profiles and
extensions are built.

---

# 2. Relationship to Other ULABI Documents

The ULABI documentation hierarchy is:

```text
ULABI-DESIGN.md
       |
       v
ULABI-SPEC.md
       |
       v
docs/abi/core-abi.md
       |
       +---- calling-convention.md
       +---- data-types.md
       +---- memory-model.md
       +---- stack-model.md
       +---- register-model.md
       +---- exception-model.md
       +---- return-values.md

ULABI-DESIGN.md defines the architecture.

ULABI-SPEC.md defines normative system-wide requirements.

This document defines the Core ABI boundary.

More specialized documents MUST NOT contradict the Core ABI.


---

3. Fundamental Rule

The fundamental rule of the Core ABI is:

> Two independently implemented components that conform to the same applicable ULABI Core ABI contract MUST agree on the representation and interpretation of the data and operations exchanged across that boundary.



The internal implementation MAY be completely different.

For example:

Language A
    |
Compiler A
    |
Runtime A
    |
    v
  ULABI
    ^
    |
Runtime B
    |
Compiler B
    |
Language B

The implementations do not need to share:

source syntax;

type-checker implementation;

garbage collector;

object model;

compiler;

runtime;

operating system.


Only the ULABI boundary contract must agree.


---

4. Zamani and Sankofa Independence

ULABI MUST remain independent from Zamani and Sankofa.

ULABI
             /     \
            /       \
       Zamani      Sankofa

Zamani MAY implement ULABI.

Sankofa MAY implement ULABI.

Neither language defines the Core ABI.

Neither language may be treated as the reference language for ULABI.

The same Core ABI MUST be implementable by unrelated languages.


---

5. Scope

The Core ABI covers:

1. ABI identity


2. ABI versioning


3. Interface identity


4. Function identity


5. Type identity


6. Primitive representations


7. Composite representations


8. Binary encoding


9. Alignment


10. Byte ordering


11. Calling conventions


12. Argument passing


13. Return values


14. Error boundaries


15. Ownership metadata


16. Lifetime metadata


17. Capability declarations


18. ABI metadata


19. Compatibility


20. Validation


21. ABI negotiation



The Core ABI does NOT mandate:

one processor calling convention;

one operating system;

one transport;

one runtime;

one memory allocator;

one garbage collector;

one object model;

one concurrency model.



---

6. Core ABI Philosophy

ULABI Core follows:

Minimal
Explicit
Deterministic
Versioned
Language-Neutral
Architecture-Neutral
Secure-by-Default
Extensible
Testable

The Core should be difficult to change.

Advanced features belong in profiles.


---

7. ABI Boundary

A ULABI boundary exists whenever independently implemented components exchange data or invoke operations under a ULABI contract.

Examples:

Language ↔ Language
Compiler ↔ Runtime
Library ↔ Application
Process ↔ Process
Application ↔ Device
Runtime ↔ Accelerator
Host ↔ WebAssembly
Machine ↔ Machine

The boundary MAY be:

in-process;

out-of-process;

local IPC;

shared memory;

remote;

hardware-mediated.


The Core contract remains conceptually stable.


---

8. Locality

Every interface SHOULD declare its locality.

Supported semantic locality classes:

LOCAL
PROCESS
HOST
REMOTE
DISTRIBUTED

A local operation MUST NOT silently become remote.

A remote operation MUST expose the additional failure and latency semantics required by the applicable profile.


---

9. ABI Identity

Every ULABI ABI implementation MUST expose an ABI identity.

The identity consists conceptually of:

ULABI
Major
Minor
Profile
Implementation ABI Mapping

Example:

ULABI/0.1/core

The exact canonical wire encoding will be finalized before ULABI 1.0.


---

10. Interface Identity

Every public ULABI interface MUST have a stable interface identifier.

Conceptually:

InterfaceID {
    namespace
    name
    version
}

The identifier MUST remain stable across compatible implementations.

The identifier MUST NOT depend on:

source-language syntax;

compiler-generated symbol names;

memory addresses;

process IDs;

machine-specific addresses.



---

11. Function Identity

Every public function MUST have a stable function identifier.

Conceptually:

FunctionID {
    InterfaceID
    FunctionName
    FunctionVersion
}

The binary representation MAY use a compact numeric identifier.

Source-language names MAY be retained as metadata but MUST NOT be the only identity mechanism.


---

12. Type Identity

Every non-primitive externally visible type SHOULD have a stable type identity.

Conceptually:

TypeID {
    namespace
    name
    version
}

A type's identity represents its ABI meaning, not its internal implementation.


---

13. Signature Identity

A function signature consists of:

FunctionID
Parameter Types
Return Type
Error Model
Calling Convention
Ownership Contract
Effect Contract

A signature change that alters binary interpretation MUST create a new compatibility boundary.


---

14. ABI Metadata

A ULABI component SHOULD expose machine-readable metadata containing:

ABI Version
Profile
Interfaces
Functions
Types
Capabilities
Dependencies
Architecture Mapping
Security Requirements

Metadata MUST itself use a versioned format.


---

15. Canonical Representation

Where ULABI defines a canonical representation, all conformant implementations MUST interpret that representation identically.

Canonical representations MUST be deterministic.

Equivalent values MUST NOT have multiple incompatible canonical encodings.


---

16. Native Representation vs ULABI Representation

ULABI distinguishes:

Source Representation
        |
        v
Implementation Representation
        |
        v
ULABI Boundary Representation

A language MAY internally represent a value differently.

For example:

Language A:
    custom_string_object

Language B:
    managed_string

ULABI:
    String {
        encoding
        length
        data
    }

Only the boundary representation must be interoperable.


---

17. Primitive Types

The Core ABI defines the following semantic primitive types:

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

Additional types MAY be introduced through future specifications.


---

18. Fixed-Width Principle

ULABI MUST prefer explicitly sized types.

ULABI MUST NOT use ambiguous concepts such as:

int
long
word
native_int
pointer-sized integer

as universal ABI types without explicitly defining their width.

This prevents architecture-dependent interpretation.


---

19. Signed Integers

Signed integers use two's-complement semantics unless a future specification explicitly defines otherwise.

Supported widths:

I8
I16
I32
I64
I128

Each type has a fixed width.

The exact binary byte ordering is defined by the applicable encoding profile.


---

20. Unsigned Integers

Unsigned integers represent values using their full available bit width.

Supported widths:

U8
U16
U32
U64
U128

Overflow behaviour MUST be explicitly defined by the operation or type contract.


---

21. Floating-Point Types

Core floating-point types:

F32
F64

A conformant implementation MUST define:

width;

encoding;

special values;

NaN behaviour;

infinity;

signed zero;

conversion behaviour.


ULABI SHOULD align with widely interoperable IEEE-compatible representations where practical.


---

22. Boolean

The semantic Boolean type has exactly two values:

false
true

ULABI MUST define a canonical boundary representation.

A receiver MUST NOT infer Boolean semantics from arbitrary memory contents.


---

23. Byte

A ULABI Byte is exactly 8 bits.

Its range is:

0..255

A byte is not necessarily text.


---

24. Character

Char represents a Unicode scalar value or the explicitly defined character domain of the applicable ULABI specification.

A language-specific character representation MUST NOT be assumed.


---

25. Unit

ULABI SHOULD define a semantic Unit type representing successful completion without a value.

Example:

Result<Unit, Error>


---

26. Strings

The Core ABI defines the semantic concept:

String

A string MUST include sufficient information to determine:

Encoding
Length
Data

ULABI SHOULD use UTF-8 as its canonical string representation.

The specification MUST define how invalid UTF-8 is handled.


---

27. String Termination

ULABI MUST NOT require null termination for strings.

A string length MUST be explicitly represented or determinable.

A null-terminated language string MAY be adapted to ULABI.


---

28. Byte Sequences

Binary data MUST be represented separately from text.

Conceptually:

Bytes {
    length
    data
}

Binary data MAY contain arbitrary byte values.


---

29. Arrays

An array consists of:

ElementType
Length
Elements

The receiver MUST be able to determine the element count.

ULABI arrays MUST NOT rely on sentinel termination.


---

30. Records

A record is an ordered or explicitly identified collection of fields.

Conceptually:

Record {
    FieldID
    FieldType
    FieldValue
}

The binary representation MUST define field ordering or explicit field identifiers.


---

31. Struct Layout

A fixed-layout struct MUST define:

Field
Type
Offset
Alignment
Size

No implementation may assume native compiler struct layout unless the ULABI profile explicitly specifies it.


---

32. Enums

An enum consists of a fixed set of variants.

Each variant MUST have a stable identifier.

Example:

Status {
    Pending
    Running
    Complete
    Failed
}

The binary discriminant representation MUST be defined.


---

33. Variants

A variant represents one of several possible data forms.

Example:

Value =
    Integer(I64)
    Text(String)
    Bytes(Bytes)

The active variant MUST be explicitly identifiable.


---

34. Option

ULABI SHOULD provide:

Option<T>

with:

None
Some(T)

The representation MUST distinguish the two states unambiguously.


---

35. Result

ULABI SHOULD provide:

Result<T, E>

with:

Ok(T)
Err(E)

The result mechanism allows languages with different error systems to interoperate.


---

36. Handles

A Handle represents an opaque reference to a resource or implementation object.

A handle MUST NOT expose implementation-specific memory layout unless the applicable contract explicitly requires it.

Example:

Handle<File>
Handle<Device>
Handle<Stream>
Handle<Process>


---

37. Opaque Objects

ULABI MAY expose opaque objects.

Opaque objects are accessed only through their published interfaces.

An implementation MUST NOT assume knowledge of the object's internal layout.


---

38. Pointer Policy

Raw native pointers MUST NOT be treated as universally portable ABI values.

A raw pointer MAY be used only within a boundary where:

address-space compatibility is guaranteed;

lifetime is defined;

ownership is defined;

safety is defined.


For portable boundaries, ULABI SHOULD prefer:

Handle
Offset
Buffer Descriptor
Capability


---

39. Buffer Descriptor

A portable buffer descriptor SHOULD contain:

Data Reference
Length
Capacity
Element Size
Alignment
Mutability
Ownership
Lifetime

The actual binary structure will be finalized by the memory-model specification.


---

40. Alignment

Every ABI-visible data structure MUST have defined alignment requirements.

Implementations MUST NOT assume alignment that the contract does not guarantee.

Misaligned data MUST be handled according to the applicable ABI rules.


---

41. Padding

ABI-visible padding MUST be deterministic where the structure is passed by value or serialized.

Padding bytes MUST NOT contain security-sensitive uninitialized data.

Canonical serialized forms SHOULD avoid unspecified padding entirely.


---

42. Size

Every fixed ABI type MUST have a defined size.

Variable-sized values MUST expose sufficient metadata to determine their encoded or allocated size.


---

43. Offset

Every field within a fixed-layout structure MUST have a determinable offset.

Offsets MUST remain stable within a stable ABI version.


---

44. Byte Ordering

ULABI SHOULD define a canonical byte ordering for portable binary representations.

A native ABI mapping MAY use native byte order internally when the boundary contract explicitly permits it.

Portable representations MUST NOT depend on native byte order.


---

45. Calling Convention

The calling convention defines how functions exchange arguments and results.

A ULABI calling convention MUST define:

Argument Order
Argument Representation
Return Representation
Register Usage
Stack Usage
Alignment
Aggregate Passing
Error Passing
Ownership Transfer

Architecture-specific details belong in architecture mappings.


---

46. Logical Calling Convention

ULABI first defines a logical calling convention:

Caller
   |
   +-- Function Identity
   +-- Arguments
   +-- Context
   +-- Capabilities
   |
   v
Callee
   |
   +-- Return Value
   +-- Error
   +-- Side Effects

The logical convention is architecture-independent.


---

47. Physical Calling Convention

A physical calling convention maps the logical convention onto a platform.

For example:

ULABI Logical Call
       |
       v
Architecture Mapping
       |
       +-- Registers
       +-- Stack
       +-- ABI-specific instructions
       +-- Alignment

The physical mapping MUST preserve the logical ULABI contract.


---

48. Argument Passing

Arguments MUST be passed according to their declared ULABI types.

The caller and callee MUST agree on:

Type
Size
Alignment
Ownership
Mutability
Lifetime


---

49. Pass-by-Value

Small fixed-size values MAY be passed by value.

The applicable calling convention MUST define the maximum value size suitable for direct passing.


---

50. Pass-by-Reference

Large or mutable values MAY be passed by reference.

The reference MUST have an explicit contract for:

Ownership
Lifetime
Mutability
Alignment
Bounds


---

51. Ownership Transfer

If a function consumes an owned value:

Caller
  |
  | ownership transfer
  v
Callee

the caller MUST NOT continue to use the resource as its own after transfer.


---

52. Borrowed Arguments

A borrowed argument does not transfer ownership.

Example:

inspect(Borrowed<Buffer>)

The callee MUST respect the declared lifetime.


---

53. Mutable Borrow

A mutable borrowed resource MUST explicitly declare exclusive mutation permission where required.

The implementation MUST prevent conflicting access according to the applicable memory profile.


---

54. Return Values

Every function MUST declare whether it returns:

No Value
One Value
Multiple Values
Result
Stream
Future
Handle

The Core ABI defines the semantic contract.

The physical representation belongs to the calling-convention mapping.


---

55. Indirect Returns

Large return values MAY be returned through caller-provided storage.

The contract MUST specify:

Who allocates
Who owns
Who initializes
Who releases
Lifetime
Alignment


---

56. Multiple Return Values

A language MAY internally support multiple return values.

At the ABI boundary, they MUST be represented as either:

Tuple
Record
Struct
Result

or another explicitly standardized representation.


---

57. Error Boundary

Errors MUST have a machine-readable representation.

An error SHOULD contain:

ErrorCode
Category
Severity
Retryability
Context
Origin

Optional information MAY include:

Cause
Stack Information
Diagnostic Data
Recovery Recommendation


---

58. Error Codes

Error codes MUST be stable within a compatible interface version.

Applications SHOULD branch on error codes rather than human-readable messages.


---

59. Error Ownership

Error objects MUST have explicit ownership and lifetime semantics.

A caller MUST NOT retain an error object beyond its declared lifetime.


---

60. Exceptions

ULABI Core does not mandate language-level exceptions.

Languages MAY translate:

ULABI Error

into:

Exception
Result
Error Value
Status Code

The binary boundary MUST remain standardized.


---

61. ABI Context

A ULABI call MAY carry execution context.

Context may include:

Caller Identity
Interface
Capabilities
Deadline
Cancellation
Tracing Context
Security Context
Locale

Context MUST be explicitly defined by the applicable profile.


---

62. Capabilities

Security-sensitive functions SHOULD declare required capabilities.

Example:

read_file(
    capability: FileRead,
    path: String
)

Possession of a function identifier MUST NOT automatically grant authority.


---

63. Capability Validation

Before performing a protected operation:

Capability
    |
    v
Validate
    |
    v
Authorize
    |
    v
Execute

Invalid capabilities MUST result in a defined security failure.


---

64. ABI Effects

A function MAY declare effects.

Possible effects:

Pure
ReadMemory
WriteMemory
ReadResource
WriteResource
Network
Filesystem
Process
Device
GPU
Time
Random
NonDeterministic

Effect metadata SHOULD be machine-readable.


---

65. Determinism

Functions MAY declare:

Deterministic
NonDeterministic
EnvironmentDependent

A function MUST NOT claim deterministic behaviour if external state can alter its result without being part of the contract.


---

66. Idempotency

A function MAY declare itself:

Idempotent
NonIdempotent
Unknown

Clients MUST NOT assume idempotency when it is not declared.

This is important for retries.


---

67. Side Effects

Side effects MUST be represented in interface metadata when they materially affect interoperability, security, or reliability.


---

68. ABI State

An interface MAY maintain state.

Stateful interfaces MUST define:

Initialization
Active State
Invalid State
Shutdown


---

69. Stateless Functions

A function that does not depend on persistent external state MAY be declared stateless.

Statelessness MAY enable:

caching;

replication;

deterministic testing;

optimization.



---

70. Lifecycle

A ULABI component SHOULD support a lifecycle:

DISCOVERED
    |
VERIFIED
    |
LOADED
    |
INITIALIZED
    |
READY
    |
RUNNING
    |
STOPPING
    |
STOPPED
    |
UNLOADED

Failures MUST transition to defined failure states.


---

71. ABI Validation

A ULABI component MUST validate ABI metadata before relying upon it.

Validation SHOULD include:

ABI Version
Profile
Interface ID
Function ID
Type ID
Size
Alignment
Capabilities
Security Requirements


---

72. Invalid ABI Data

Malformed ABI data MUST NOT result in undefined memory access.

Possible responses include:

InvalidABI
InvalidType
InvalidLength
InvalidAlignment
UnsupportedVersion
UnsupportedFeature
SecurityViolation


---

73. Length Validation

Variable-length values MUST be bounds-checked.

An implementation MUST NOT trust externally supplied lengths without validation.


---

74. Offset Validation

Offsets MUST be validated before memory access.

An offset outside the permitted object or buffer MUST result in an error.


---

75. Integer Conversion

Conversions between ULABI numeric types MUST explicitly define:

Overflow
Underflow
Truncation
Sign Conversion
Precision Loss

Silent unsafe conversion MUST NOT occur across a boundary.


---

76. Type Compatibility

Two types are compatible only when their ULABI semantics and binary representation satisfy the applicable compatibility rules.

Matching source-language names are insufficient.


---

77. Structural Compatibility

Two structures MAY be structurally compatible if:

Fields
Types
Ordering/Layout
Alignment
Optionality
Ownership

match the applicable compatibility rules.


---

78. Semantic Compatibility

Binary compatibility alone is not sufficient.

Two interfaces MUST also preserve required semantics.

For example:

Function A:
    deletes resource

Function B:
    only reads resource

They are not semantically compatible even if their binary signatures are identical.


---

79. Version Compatibility

ULABI uses versioned compatibility.

A compatible implementation MUST identify:

Major
Minor
Patch

or the equivalent profile-defined version.


---

80. Major Version

A major version change MAY introduce incompatible changes.

An implementation MUST NOT assume major-version compatibility.


---

81. Minor Version

A minor version SHOULD preserve backward compatibility within the same major version.

New optional functionality SHOULD be introduced through minor versions or extensions.


---

82. Patch Version

Patch versions SHOULD contain:

fixes;

clarifications;

security improvements;

implementation corrections.


Patch versions MUST NOT intentionally alter the ABI contract.


---

83. ABI Negotiation

Before using optional functionality:

Discover
   |
Compare
   |
Negotiate
   |
Verify
   |
Use

An implementation MUST NOT assume optional capabilities.


---

84. Capability Discovery

A component SHOULD expose:

Supported Profiles
Supported Interfaces
Supported Versions
Supported Types
Supported Extensions


---

85. Unknown Features

Unknown optional features SHOULD be ignored.

Unknown mandatory features MUST cause explicit incompatibility.

An implementation MUST NOT silently reinterpret an unknown mandatory feature.


---

86. Compatibility Matrix

A component MAY expose a machine-readable compatibility matrix:

Feature              Supported
--------------------------------
Core ABI             Yes
Types                Yes
Memory               Yes
Async                No
Security             Yes
Distributed          No
Self-Healing         No


---

87. ABI Negotiation Failure

Negotiation failure MUST produce an explicit result.

Example:

UnsupportedVersion
UnsupportedProfile
UnsupportedType
UnsupportedCapability
IncompatibleSignature
SecurityMismatch


---

88. ABI Mapping

ULABI Core is architecture-neutral.

A platform implementation uses:

ULABI Core
    |
    v
Architecture Mapping
    |
    v
Native ABI

The mapping may target:

x86-64
ARM64
ARM32
RISC-V
WASM
Other architectures

The list is extensible.


---

89. Native ABI Interoperability

ULABI MAY map to an existing native ABI.

For example:

ULABI
  |
  +-- Native C ABI
  |
  +-- Native platform ABI
  |
  +-- Custom ABI

The native ABI MUST NOT redefine the ULABI semantics.


---

90. Architecture Mapping Requirements

Every architecture mapping MUST specify:

Register Rules
Stack Rules
Alignment
Argument Passing
Return Values
Aggregate Passing
Calling Convention
Exception Boundary
Thread Context

These details belong in architecture-specific specifications.


---

91. Register Independence

The Core ABI MUST NOT require a particular CPU register.

Register assignments belong to architecture mappings.


---

92. Stack Independence

The Core ABI MUST NOT assume a universal native stack layout.

The logical ABI semantics must remain independent of physical stack design.


---

93. Zero-Copy

Zero-copy is OPTIONAL.

When zero-copy is used, the implementation MUST preserve:

Ownership
Lifetime
Bounds
Alignment
Mutability
Security

Zero-copy optimization MUST NOT weaken safety.


---

94. Copy-Based Fallback

A ULABI implementation SHOULD provide a copy-based fallback where zero-copy is unavailable.

This allows the same logical interface to operate across different environments.


---

95. Streaming

Streaming is an extension.

The Core ABI MUST NOT require all values to fit into memory at once.

A future streaming profile SHOULD define:

Stream Identity
Element Type
Chunking
Backpressure
Completion
Cancellation
Failure


---

96. Large Data

Large data SHOULD be transferable through:

Buffers
Handles
Streams
Shared Memory
Files
Object References

rather than requiring enormous stack arguments.


---

97. Resource Handles

Resource handles MUST identify resources without exposing internal implementation details.

Examples:

FileHandle
DeviceHandle
MemoryHandle
StreamHandle
ProcessHandle
GPUHandle


---

98. Handle Lifetime

A handle MUST have explicit lifetime semantics.

Invalid or expired handles MUST produce defined errors.


---

99. Handle Security

A handle MUST NOT automatically grant broader privileges than the capability under which it was created.


---

100. Thread Safety

Thread safety is an interface property.

A ULABI interface SHOULD declare:

ThreadSafe
ThreadCompatible
ThreadConfined
NotThreadSafe


---

101. Reentrancy

A ULABI interface SHOULD declare whether calls may be reentrant.

Callbacks MUST NOT assume non-reentrancy unless the contract explicitly states it.


---

102. Async Boundary

Asynchronous execution belongs primarily to the Async Profile.

However, the Core ABI MUST allow an interface to declare that an operation is not synchronous.


---

103. Callback Identity

Callbacks MUST have stable interface and function identities.

Callback lifetimes MUST be explicitly defined.


---

104. Callback Safety

A callback MUST NOT be invoked after its lifetime expires.

If callback cancellation exists, cancellation semantics MUST be defined.


---

105. ABI Events

An implementation MAY expose ABI events:

Created
Started
Stopped
Failed
Recovered
Changed
Disconnected
Reconnected

Events MUST use versioned schemas.


---

106. ABI Introspection

Authorized tooling SHOULD be able to inspect:

Interfaces
Functions
Types
Versions
Capabilities
Memory Contracts
Effects

Introspection MUST respect security policies.


---

107. Debugging

ULABI-aware debugging SHOULD expose the logical ABI representation rather than only the native implementation representation.

Example:

ULABI:
    I64 value = 42

Native:
    architecture-specific representation


---

108. ABI Tracing

ULABI tracing MAY record:

Interface
Function
Arguments Metadata
Result
Error
Duration
Capability
Correlation ID

Sensitive values SHOULD be redacted according to policy.


---

109. ABI Security

The Core ABI MUST assume that ABI boundaries can be security boundaries.

Implementations SHOULD protect against:

malformed metadata;

type confusion;

integer overflow;

buffer overflow;

use-after-free;

invalid handles;

capability abuse;

confused deputy attacks;

replay where applicable.



---

110. Type Confusion

A component MUST NOT interpret a value as a different ULABI type merely because the binary sizes happen to match.

For example:

I64 != Handle
I64 != Pointer
Bytes != String

unless an explicit conversion exists.


---

111. Memory Safety

The ABI boundary MUST preserve memory safety.

Invalid external values MUST NOT permit unauthorized memory access.


---

112. Capability Safety

A function requiring a capability MUST verify that capability before performing the protected operation.


---

113. Resource Exhaustion

Implementations SHOULD impose resource limits.

Potential limits include:

Memory
CPU
Handles
Message Size
Call Depth
Streams
Execution Time


---

114. Call Depth

Implementations SHOULD protect against unbounded recursive ABI calls.

A call-depth limit MAY produce:

ResourceExhausted

or an equivalent error.


---

115. ABI Reentrancy Attacks

Security-sensitive implementations SHOULD account for reentrant callbacks and unexpected nested calls.

State MUST NOT be left in an invalid security state.


---

116. Deterministic Encoding

Canonical ABI encodings SHOULD be deterministic.

Given the same logical value and same encoding profile:

Value A
   |
Encoder
   |
Bytes X

must produce the same canonical byte sequence.


---

117. Canonical Decoding

A decoder MUST reject malformed canonical encodings.

Where multiple encodings are permitted by an extension, the extension MUST define equivalence rules.


---

118. ABI Serialization Boundary

Serialization is separate from local function calling.

The Core ABI SHOULD define shared semantic types while allowing separate serialization profiles.

This prevents the Core from becoming dependent on one network protocol.


---

119. Distributed ABI

Distributed ABI functionality belongs to a separate profile.

A distributed implementation MUST account for:

Latency
Timeout
Network Failure
Authentication
Authorization
Serialization
Partial Failure
Retry
Cancellation


---

120. Failure Semantics

A ULABI function MUST have defined failure behaviour.

Failure MUST NOT leave the ABI boundary in an unspecified state.


---

121. Atomicity

A function MAY declare itself atomic.

If not declared atomic, callers MUST NOT assume that partial execution can be rolled back.


---

122. Transaction Boundaries

Transactions belong to an extension profile.

If transactional behaviour is exposed, the interface MUST define:

Begin
Prepare
Commit
Rollback

where applicable.


---

123. Self-Healing Boundary

Self-healing belongs to the Reliability / Self-Healing Profile.

The Core ABI only provides the stable primitives necessary to represent:

Health
Failure
State
Error
Recovery Result

The Core MUST NOT require autonomous self-modification.


---

124. Safe Recovery Principle

Any self-healing implementation built on ULABI MUST obey:

Detect
  ↓
Diagnose
  ↓
Policy Check
  ↓
Recover
  ↓
Verify
  ↓
Healthy

If recovery fails:

Rollback
  ↓
Escalate


---

125. No Unauthorized Self-Modification

ULABI does not authorize arbitrary code modification.

A self-healing implementation MUST operate within explicit policy and capability boundaries.


---

126. Conformance

A Core ABI implementation is conformant only if it passes the applicable Core ABI conformance tests.

At minimum, tests SHOULD cover:

Identity
Version
Primitive Types
Composite Types
Function Calls
Arguments
Returns
Errors
Ownership
Alignment
Encoding
Validation
Compatibility


---

127. Cross-Language Conformance

At least two independently implemented language bindings SHOULD be tested against the same Core ABI contract.

Example:

Language A
    |
    v
ULABI Core
    ^
    |
Language B

The test MUST verify actual interoperability.


---

128. Cross-Architecture Conformance

Where an architecture mapping exists, conformance SHOULD be tested on more than one processor architecture.


---

129. Negative Testing

Conformance tests MUST include invalid cases.

Examples:

Invalid Type
Invalid Version
Invalid Length
Invalid Offset
Invalid Alignment
Invalid Handle
Invalid Capability
Malformed Encoding
Unsupported Feature

The implementation MUST fail safely.


---

130. Fuzz Testing

Core ABI parsers SHOULD be fuzz tested.

Particular targets:

Type Descriptors
Interface Metadata
Function Signatures
Serialized Values
Handles
Lengths
Offsets
Version Information


---

131. ABI Differential Testing

Different implementations SHOULD be tested against each other.

Example:

Implementation A
       |
       +---- encode ----+
                        |
                        v
                 Implementation B
                        |
                        +---- decode

The decoded semantic value MUST match the original.


---

132. ABI Round-Trip Testing

The following property SHOULD hold:

decode(encode(value)) == value

for all values permitted by the applicable type contract.


---

133. ABI Compatibility Testing

For every compatible version:

Old Producer → New Consumer
New Producer → Old Consumer

SHOULD be tested where forward and backward compatibility are claimed.


---

134. ABI Difference Detection

Tooling SHOULD be able to compare two interface definitions:

ULABI Interface v1
        |
        v
ABI Diff
        |
        v
ULABI Interface v2

The tool SHOULD identify:

breaking changes;

compatible additions;

removed functions;

changed types;

changed ownership;

changed effects;

changed alignment;

changed calling conventions.



---

135. ABI Registry

ULABI MAY provide a registry for:

Interface IDs
Type IDs
Version Information
Profiles
Extensions
Capability Names
Error Codes

Registry infrastructure MUST NOT become a mandatory runtime dependency.


---

136. Namespace Rules

Public identifiers SHOULD use namespaces to avoid collisions.

Example:

org.example.calculator

ULABI itself SHOULD reserve its own namespace.


---

137. Vendor Extensions

Vendors MAY define extensions.

Vendor extensions MUST NOT alter Core semantics.

Extensions SHOULD use separate namespaces.


---

138. Experimental Features

Experimental ABI features MUST be explicitly marked.

Experimental features MUST NOT be presented as stable ULABI Core.


---

139. ABI Stability

The Core ABI MUST be treated as a long-term compatibility contract.

Changes should be conservative.

Before changing a Core feature, the project SHOULD demonstrate:

Use Cases
Implementation Experience
Cross-Language Testing
Security Review
Compatibility Analysis


---

140. Core Minimalism

A feature SHOULD enter the Core only if it is:

Universal
Necessary
Stable
Implementable
Testable
Language-Neutral
Architecture-Neutral

Otherwise it SHOULD remain an extension.


---

141. Implementation Freedom

ULABI does not dictate implementation strategy.

An implementation MAY use:

AOT Compiler
JIT Compiler
Interpreter
Virtual Machine
Native Runtime
Garbage Collection
Reference Counting
Manual Memory Management
Ownership Checking
Hybrid Runtime

provided that the ULABI contract is preserved.


---

142. Reference Implementation

A reference implementation MAY be created.

It MUST serve as:

Example
Test Target
Interoperability Baseline

It MUST NOT become the definition of ULABI.

The specification remains authoritative.


---

143. Reference ABI Architecture

A conceptual reference implementation may use:

+--------------------------------------+
|           ULABI Interface            |
+--------------------------------------+
|       Metadata / Type System         |
+--------------------------------------+
|       Validation / Compatibility     |
+--------------------------------------+
|       Memory / Ownership Layer       |
+--------------------------------------+
|       Calling Convention Layer       |
+--------------------------------------+
|       Architecture Adapter           |
+--------------------------------------+
|       Native Runtime / OS            |
+--------------------------------------+


---

144. Core ABI Invariants

The following invariants MUST hold.

Invariant 1 — Identity

Two different interfaces MUST NOT intentionally share the same stable interface identifier.

Invariant 2 — Type Safety

A value MUST NOT be interpreted as an incompatible type.

Invariant 3 — Bounds Safety

An ABI-provided length or offset MUST NOT permit an out-of-bounds access.

Invariant 4 — Ownership

A component MUST NOT release a resource it does not own.

Invariant 5 — Lifetime

A component MUST NOT access an expired resource.

Invariant 6 — Capability

A protected operation MUST NOT execute without the required authority.

Invariant 7 — Version

An incompatible version MUST NOT silently be treated as compatible.

Invariant 8 — Determinism

Canonical encoding MUST be deterministic.

Invariant 9 — Independence

Core semantics MUST NOT depend on a particular programming language.

Invariant 10 — Architecture Neutrality

Core semantics MUST NOT depend on a particular processor architecture.


---

145. Core ABI State Machine

A component implementing the Core ABI SHOULD follow:

+-------------+
             | DISCOVERED  |
             +------+------+
                    |
                    v
             +-------------+
             |  VERIFIED   |
             +------+------+
                    |
                    v
             +-------------+
             |   LOADED    |
             +------+------+
                    |
                    v
             +-------------+
             | INITIALIZED |
             +------+------+
                    |
                    v
             +-------------+
             |    READY    |
             +------+------+
                    |
                    v
             +-------------+
             |   RUNNING   |
             +------+------+
                    |
          +---------+---------+
          |                   |
          v                   v
      STOPPING             FAILED
          |                   |
          v                   v
       STOPPED             RECOVERY
                              |
                         +----+----+
                         |         |
                         v         v
                      HEALTHY   ESCALATED


---

146. ABI Call Lifecycle

A normal call follows:

Caller
  |
  v
Resolve Interface
  |
  v
Validate Version
  |
  v
Validate Signature
  |
  v
Validate Capabilities
  |
  v
Validate Arguments
  |
  v
Invoke
  |
  v
Validate Return
  |
  v
Transfer Ownership
  |
  v
Return


---

147. ABI Failure Lifecycle

Call
 |
 v
Validation
 |
 +---- invalid ----> Error
 |
 v
Execution
 |
 +---- failure ----> Error
 |
 v
Return

A failure MUST NOT silently become an unrelated success.


---

148. ABI Security Lifecycle

Discover
   |
Verify Identity
   |
Verify Integrity
   |
Verify Version
   |
Verify Capability
   |
Verify Policy
   |
Execute


---

149. ABI Compatibility Lifecycle

Discover
   |
Read ABI Metadata
   |
Compare Versions
   |
Compare Profiles
   |
Compare Types
   |
Compare Functions
   |
Compare Security
   |
Compatible?
   |
 +--+--+
 |     |
Yes    No
 |     |
Use   Reject


---

150. ABI Contract Example

Conceptual interface:

interface Calculator {

    add(
        left: I64,
        right: I64
    ) -> Result<I64, Error>;

    subtract(
        left: I64,
        right: I64
    ) -> Result<I64, Error>;
}

The source-language implementation might be:

Language A:
    add(a, b)

Language B:
    add(x, y)

The names of local parameters do not matter.

The ULABI contract does.


---

151. Memory Example

Conceptual interface:

create_buffer(
    size: U64
) -> Result<Owned<Buffer>, Error>;

The caller owns the resulting buffer.

The implementation MUST provide a valid lifetime and release mechanism.


---

152. Borrowing Example

inspect(
    buffer: Borrowed<Buffer>
) -> Result<Inspection, Error>;

The function may inspect the buffer.

It MUST NOT:

free it;

retain it beyond the permitted lifetime;

modify it without permission.



---

153. Capability Example

read(
    capability: FileRead,
    file: Handle<File>
) -> Result<Bytes, Error>;

The handle and capability are separate concepts.

A handle identifies a resource.

The capability authorizes an operation.


---

154. Async Example

A future Async Profile may expose:

calculate_async(
    input: I64
) -> Future<Result<I64, Error>>;

The Core ABI does not require one particular future implementation.


---

155. Cross-Language Example

+-------------+
| Language A  |
+------+------+
       |
       | adapter
       v
+-------------+
| ULABI Core  |
+------+------+
       ^
       | adapter
       |
+------+------+
| Language B  |
+-------------+

Language A and Language B remain independent.


---

156. Cross-Process Example

Process A
    |
    | ULABI IPC
    |
Process B

The processes MAY use completely different runtimes.


---

157. Cross-Machine Example

Machine A
    |
ULABI Distributed Profile
    |
Transport
    |
Machine B

The distributed profile adds networking semantics without changing the fundamental Core ABI model.


---

158. Hardware Example

Application
    |
ULABI
    |
Hardware Capability
    |
GPU / NPU / FPGA / Accelerator

Hardware-specific details belong to hardware profiles.


---

159. Embedded Example

Embedded Application
        |
      ULABI
        |
    Device Driver
        |
      Hardware

The Core should remain usable even where no traditional operating system exists.


---

160. Future Architecture Example

ULABI should allow future architectures to define:

ULABI Core
     |
Future Architecture Mapping
     |
Native Execution

without modifying the semantic Core.


---

161. What the Core ABI Must Never Do

The Core ABI MUST NOT:

require one programming language;

require one compiler;

require one runtime;

require one operating system;

require one processor;

require one company;

require one transport;

require one memory-management model;

require one garbage collector;

require one object model.



---

162. What the Core ABI Must Provide

The Core ABI MUST provide the foundation for:

Identity
Types
Calls
Data
Errors
Memory Contracts
Versioning
Compatibility
Validation
Security Boundaries


---

163. Implementation Priority

The first implementation SHOULD build the Core in this order:

1. ABI Metadata
       |
2. Interface Identity
       |
3. Type Identity
       |
4. Primitive Types
       |
5. Composite Types
       |
6. Canonical Encoding
       |
7. Function Signatures
       |
8. Calling Convention
       |
9. Arguments
       |
10. Return Values
       |
11. Errors
       |
12. Ownership
       |
13. Validation
       |
14. Version Negotiation
       |
15. Compatibility Testing

Only after these are working should advanced profiles become implementation priorities.


---

164. Required Companion Specifications

The following documents MUST provide deeper definitions:

docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

These documents MUST refine the Core ABI rather than redefine it.


---

165. Conformance Checklist

A Core ABI implementation SHOULD be evaluated against:

[ ] ABI Identity
[ ] Interface Identity
[ ] Function Identity
[ ] Type Identity
[ ] Primitive Types
[ ] Composite Types
[ ] Canonical Encoding
[ ] Decoding
[ ] Alignment
[ ] Size Rules
[ ] Offset Rules
[ ] Calling Convention
[ ] Arguments
[ ] Return Values
[ ] Error Model
[ ] Ownership
[ ] Lifetimes
[ ] Capability Metadata
[ ] Validation
[ ] Versioning
[ ] Feature Negotiation
[ ] Compatibility
[ ] Security
[ ] Negative Testing
[ ] Cross-Language Testing


---

166. Minimum Core Profile

The initial ULABI Core Profile SHOULD require only:

Identity
Versioning
Primitive Types
Basic Composite Types
Function Contracts
Arguments
Return Values
Errors
Canonical Representation
Validation
Compatibility Metadata

Memory, asynchronous execution, distributed operation, self-healing, hardware acceleration, and advanced security SHOULD remain profile-driven where practical.


---

167. Core ABI Design Rule

When there is a choice between:

More Core Features

and:

Smaller Stable Core

ULABI SHOULD prefer the smaller stable Core unless the feature is essential to universal interoperability.


---

168. Long-Term Goal

The long-term goal is:

ULABI
                   |
      +------------+------------+
      |            |            |
  Language A   Language B   Language C
      |            |            |
      +------------+------------+
                   |
          Common ABI Contract

An ecosystem should eventually be able to add a new programming language without redesigning existing ULABI interfaces.


---

169. Ultimate Interoperability Property

The strongest test of the Core ABI is:

New Language
     |
New Compiler
     |
New Runtime
     |
ULABI Adapter
     |
Existing ULABI Ecosystem

If a new language can join the ecosystem without changing the existing languages, ULABI is achieving its intended purpose.


---

170. Status

This document is a:

DRAFT CORE ABI DESIGN

It is not yet a frozen binary standard.

Before ULABI 1.0, the following MUST be finalized through implementation, testing, review, and interoperability experiments:

1. Canonical binary encoding


2. Exact primitive encodings


3. Exact composite layouts


4. Calling conventions


5. Architecture mappings


6. Type descriptor format


7. ABI metadata format


8. Interface identifier format


9. Error encoding


10. Ownership representation


11. Capability representation


12. Version negotiation


13. Conformance requirements



No implementation should treat unresolved draft details as permanently frozen.


---

171. Final Principle

ULABI Core exists to make this possible:

ANY LANGUAGE
               |
               v
          ULABI ADAPTER
               |
               v
        ┌───────────────┐
        │   ULABI CORE  │
        └───────────────┘
               |
               v
          ANY LANGUAGE

The language changes.

The compiler changes.

The runtime changes.

The operating system changes.

The hardware changes.

The implementation changes.

The universal interoperability contract remains.


---

END OF docs/abi/core-abi.md

**Design 3 is deliberately more technical than `ULABI-DESIGN.md` and `ULABI-SPEC.md`.** The next logical document is **Design 4: `docs/abi/calling-convention.md`**, where we can define how ULABI calls actually map arguments, registers, stack frames, aggregates, pointers/handles, returns, variadic calls, callbacks, and architecture-specific ABIs without making ULABI dependent on any particular CPU.