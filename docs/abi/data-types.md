I read the current ULABI-DESIGN.md in Benwellonedge28/ULABI first. It establishes the Minimal Core + Profiles architecture, language/runtime/platform neutrality, semantic types, canonical representations, memory boundaries, security, self-healing, distributed operation, hardware profiles, compatibility, and conformance direction.

I also checked the repository structure: docs/abi/core-abi.md and docs/abi/data-types.md already exist, along with ULABI-SPEC.md.

Therefore Design Document 5 should be docs/abi/data-types.md, and it should be written as a complete contract rather than a simple description.

Below is the complete file you can copy and paste.

# ULABI Universal Data Types Specification

**Document:** `docs/abi/data-types.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Specification Area:** Universal ABI Data Types  
**Version:** 0.1.0-draft  
**Status:** Design Specification / Pre-Implementation  
**License:** Apache-2.0  

---

# 1. Purpose

This document defines the universal data-type model used by ULABI.

The purpose of the ULABI data-type system is to provide a stable semantic contract through which independently implemented programming languages, compilers, runtimes, libraries, applications, operating systems, devices, and distributed systems can exchange values.

The ULABI data-type system MUST:

- remain language-neutral;
- remain compiler-neutral;
- remain runtime-neutral;
- remain operating-system-neutral;
- remain architecture-neutral;
- remain vendor-neutral;
- distinguish semantic meaning from implementation representation;
- provide deterministic boundary representations;
- support safe cross-language conversion;
- support versioned evolution;
- support compatibility analysis;
- support validation;
- support security boundaries;
- support local and distributed interoperability;
- support future extensions without destabilizing the Core.

ULABI data types define what a value means and how that value can safely cross a ULABI boundary.

They do not require participating languages to represent that value internally in the same way.

---

# 2. Relationship to the ULABI Architecture

The ULABI architecture is based on:

```text
Language
    |
Compiler
    |
Runtime
    |
Adapter
    |
    v
+----------------------+
|      ULABI           |
|                      |
| Semantic Contract    |
| Type Contract        |
| ABI Contract         |
| Error Contract       |
+----------------------+
    |
Adapter
    |
Runtime
    |
Compiler
    |
Language

The data-type specification is one layer of the Core ABI.

The documentation relationship is:

ULABI-DESIGN.md
       |
       v
ULABI-SPEC.md
       |
       v
docs/abi/core-abi.md
       |
       v
docs/abi/data-types.md
       |
       +-----------------------------+
       |                             |
       v                             v
memory-model.md              calling-convention.md
       |
       +-----------------------------+
       |
       v
type-system specifications

ULABI-DESIGN.md defines the overall architecture.

ULABI-SPEC.md defines system-wide normative requirements.

docs/abi/core-abi.md defines the fundamental ABI boundary.

This document defines the universal data-type contract.

Specialized documents MAY extend this type system, but MUST NOT contradict the rules established here.


---

3. Fundamental Principle

The fundamental principle is:

> ULABI defines semantic types at the interoperability boundary, not the internal type system of any programming language.



For example:

Language A
    |
    | custom internal representation
    v
ULABI String
    ^
    | different internal representation
    |
Language B

Language A and Language B do not need to use the same internal representation.

They only need to agree on the ULABI meaning and boundary representation.


---

4. Zamani and Sankofa Independence

ULABI MUST remain independent from both Zamani and Sankofa.

ULABI
                /     \
               /       \
          Zamani      Sankofa

Zamani is one independent programming language.

Sankofa is another independent programming language.

Neither language defines ULABI.

Neither language is the reference language for ULABI.

Neither language may impose its internal type system upon ULABI.

Both MAY implement ULABI adapters.

The same ULABI type contract MUST be implementable by unrelated languages.

This principle applies equally to:

C;

C++;

Rust;

Go;

Java;

Python;

C#;

Swift;

Kotlin;

Fortran;

Ada;

JavaScript;

TypeScript;

WebAssembly environments;

future programming languages.



---

5. Type-System Goals

The ULABI data-type system SHALL provide:

1. Semantic stability.


2. Binary stability.


3. Explicit representation.


4. Explicit width.


5. Explicit encoding.


6. Explicit ownership.


7. Explicit lifetime.


8. Explicit mutability.


9. Explicit nullability.


10. Explicit validity.


11. Explicit bounds.


12. Explicit alignment where applicable.


13. Deterministic encoding.


14. Version-aware evolution.


15. Compatibility analysis.


16. Security validation.


17. Cross-language conversion.


18. Cross-platform operation.


19. Distributed serialization.


20. Streaming support.


21. Zero-copy support where safe.


22. Extensibility.


23. Formal verification opportunities.


24. Conformance testing.




---

6. Type Categories

ULABI divides types into the following categories:

ULABI Types
    |
    +-- Primitive Types
    |
    +-- Scalar Types
    |
    +-- Text Types
    |
    +-- Binary Types
    |
    +-- Collection Types
    |
    +-- Structured Types
    |
    +-- Algebraic Types
    |
    +-- Reference Types
    |
    +-- Resource Types
    |
    +-- Capability Types
    |
    +-- Temporal Types
    |
    +-- Numeric Extension Types
    |
    +-- Compute Types
    |
    +-- Stream Types
    |
    +-- Future / Async Types
    |
    +-- Extension Types

The Core MUST remain minimal.

Not every category must be part of the mandatory Core.

Advanced categories MAY be defined through ULABI profiles.


---

7. Type Identity

Every externally visible non-primitive type SHOULD have a stable type identifier.

Conceptually:

TypeID {
    namespace
    name
    version
}

Example:

com.example.finance/Money/1

Type identity MUST NOT depend solely upon:

source-language spelling;

compiler-generated names;

memory addresses;

process IDs;

file locations;

object addresses.


A type identifier MUST remain stable across compatible implementations.


---

8. Type Descriptor

A ULABI type MAY be described by machine-readable metadata.

Conceptually:

TypeDescriptor {
    type_id
    kind
    version
    flags
    size
    alignment
    encoding
    ownership
    lifetime
    mutability
    nullability
    constraints
}

The exact binary descriptor format SHALL be defined by the future schema/metadata specification.


---

9. Primitive Types

The initial Core primitive types are:

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

Byte
Char

Unit

Fixed-width types are preferred over architecture-dependent types.


---

10. Boolean

ULABI Boolean has exactly two semantic states:

false
true

A Boolean MUST NOT contain an additional semantic state.

The canonical representation SHALL be defined by the canonical encoding specification.

A receiver MUST NOT infer Boolean semantics from arbitrary memory contents.


---

11. Byte

A ULABI Byte is exactly 8 bits.

Valid values:

0..255

Byte is binary data.

Byte MUST NOT automatically be interpreted as text.


---

12. Signed Integers

ULABI signed integers use fixed widths:

I8
I16
I32
I64
I128

The semantic representation SHALL use two's-complement arithmetic unless a future normative specification explicitly defines another representation.

Each width has a fixed value range.

For example:

I8
    -128 .. 127

I16
    -32768 .. 32767

I32
    -2147483648 .. 2147483647

Implementations MUST NOT silently reinterpret one signed width as another.


---

13. Unsigned Integers

ULABI unsigned integer types are:

U8
U16
U32
U64
U128

Each type has a fixed width.

Unsigned arithmetic MUST have explicitly defined overflow behavior.

Overflow MUST NOT depend on the implementation language.


---

14. Architecture-Dependent Integer Types

ULABI MUST NOT use ambiguous universal types such as:

int
long
word
native_int
pointer-sized integer

as portable ABI types.

An implementation MAY expose architecture-specific aliases through an architecture profile.

For example:

ULABI.NativeInt

MUST resolve explicitly to a defined width within that profile.


---

15. Floating-Point Types

The Core floating-point types are:

F32
F64

The specification SHALL define:

width;

representation;

NaN;

positive infinity;

negative infinity;

signed zero;

precision;

conversion;

comparison semantics.


Where practical, ULABI SHOULD use widely interoperable IEEE-compatible representations.


---

16. Floating-Point NaN

ULABI MUST define how NaN values are represented at the boundary.

The implementation MUST NOT assume that all NaN bit patterns are semantically distinct.

Canonical encodings SHOULD normalize NaN representations where required.


---

17. Floating-Point Infinity

ULABI SHALL distinguish:

+Infinity
-Infinity

Infinity MUST NOT be silently converted into an ordinary finite value.


---

18. Floating-Point Signed Zero

ULABI SHALL preserve the semantic distinction where required:

+0
-0

Operations that intentionally normalize signed zero MUST declare that behavior.


---

19. Character

Char represents a Unicode scalar value.

ULABI MUST NOT assume that:

Char == 8 bits

or:

Char == 16 bits

or:

Char == 32 bits

Internally, languages MAY use different character representations.

The ULABI boundary representation MUST be explicit.


---

20. String

ULABI defines:

String

as a sequence of Unicode text.

The canonical text encoding SHOULD be UTF-8.

A String boundary contract MUST provide enough information to determine:

encoding;

length;

data;

validity.


Conceptually:

String {
    encoding
    length
    data
}


---

21. String Length

String length MUST have explicitly defined semantics.

ULABI SHOULD distinguish:

byte length
code-point length
grapheme length

These MUST NOT be confused.

The canonical ABI length SHOULD represent encoded byte length.

Higher-level character counts MAY be exposed separately.


---

22. Invalid Strings

ULABI MUST define behavior for invalid encoded strings.

A conformant implementation MUST NOT silently reinterpret invalid text as valid text unless the applicable contract explicitly specifies a recovery policy.

Possible policies include:

Reject
Replace
Escape
Binary fallback

The policy MUST be explicit.


---

23. Null-Termination

ULABI MUST NOT require null-terminated strings.

A language that uses:

"hello\0"

internally MAY adapt that string to ULABI.

The ULABI String contract is length-aware.


---

24. Bytes

Binary data is represented by:

Bytes

Conceptually:

Bytes {
    length
    data
}

Bytes MAY contain arbitrary byte values.

Bytes MUST remain semantically distinct from String.

String != Bytes


---

25. Arrays

An array is an ordered fixed or dynamically sized sequence of elements of one declared type.

Conceptually:

Array<T> {
    element_type
    length
    elements
}

The receiver MUST be able to determine the element count.

Sentinel termination MUST NOT be required.


---

26. Lists

ULABI MAY define:

List<T>

as a dynamically sized ordered collection.

Unlike an ABI-visible native array, a List MAY carry additional metadata such as:

length
capacity
ownership
mutability
allocator information

The exact representation belongs to the collection specification.


---

27. Maps

ULABI MAY define:

Map<K,V>

A Map specification MUST define:

valid key types;

key equality;

duplicate-key behavior;

ordering semantics;

canonical encoding;

iteration semantics;

nullability;

mutation behavior.


If ordering is not guaranteed, the specification MUST say so explicitly.


---

28. Sets

ULABI MAY define:

Set<T>

A Set MUST define:

uniqueness;

equality;

ordering;

canonical representation;

insertion/removal behavior.


A Set MUST NOT rely on implementation-specific hash functions for semantic identity.


---

29. Tuples

ULABI MAY support:

Tuple<T1,T2,...,TN>

Tuple element order is significant.

For example:

Tuple<String, I32>

is not semantically identical to:

Tuple<I32, String>


---

30. Records

A Record is a structured collection of named or identified fields.

Example:

Person {
    name: String
    age: U32
    active: Bool
}

Every ABI-visible field MUST have a stable identity.

The record contract MUST define:

field identifier;

field type;

required/optional status;

default behavior;

compatibility behavior;

unknown-field behavior.



---

31. Structs

A fixed-layout structure MAY be represented as:

Struct {
    field
    type
    offset
    alignment
    size
}

A ULABI implementation MUST NOT assume native compiler struct layout unless an applicable architecture profile explicitly defines it.


---

32. Struct Evolution

A structure MUST be designed for version evolution.

Possible evolution:

Version 1

Person {
    name
    age
}

Version 2

Person {
    name
    age
    email
}

Adding an optional field MAY preserve compatibility.

Changing the semantic meaning of an existing field MUST NOT be treated as a compatible change.


---

33. Enumerations

ULABI Enums represent a finite set of variants.

Example:

Status {
    Pending
    Running
    Complete
    Failed
}

Every variant MUST have a stable identifier.

The discriminant representation MUST be defined.


---

34. Enum Evolution

Adding an enum variant MAY be backward compatible only when the receiver has explicitly defined behavior for unknown variants.

Otherwise, the change requires a new compatibility boundary.


---

35. Variants

A Variant represents one of multiple possible data forms.

Example:

Value =
    Integer(I64)
    Text(String)
    Binary(Bytes)

The active variant MUST be explicitly identifiable.

A receiver MUST NOT infer the active variant from arbitrary memory contents.


---

36. Option

ULABI SHOULD provide:

Option<T>

with:

None
Some(T)

The representation MUST distinguish the states unambiguously.

None MUST NOT be confused with:

zero;

empty string;

empty bytes;

null pointer;

invalid value.



---

37. Result

ULABI SHOULD provide:

Result<T,E>

with:

Ok(T)
Err(E)

This provides a language-neutral error/result boundary.

An implementation MAY map this to:

exceptions;

error codes;

tagged unions;

algebraic data types;

status objects;

native result types.


The ULABI boundary remains explicit.


---

38. Error Type

An error MAY be represented as:

Error {
    code
    category
    message
    details
    cause
}

The complete error ABI SHALL be defined by the exception/error specification.

Error values MUST NOT rely solely on human-readable messages for machine interpretation.


---

39. Handles

A Handle represents an opaque reference to an implementation-managed resource.

Examples:

Handle<File>
Handle<Device>
Handle<Stream>
Handle<Process>
Handle<Thread>
Handle<Memory>

A Handle MUST NOT expose private implementation memory.

A Handle MUST have defined:

identity;

lifetime;

ownership;

access rights;

invalidation behavior.



---

40. Resources

A Resource is an externally managed object with lifecycle semantics.

Examples:

File
Socket
Device
GPUContext
Process
Thread
MemoryRegion
Stream

Resource types SHOULD normally be represented through Handles or capabilities.


---

41. Capabilities

A Capability represents authority to perform a permitted operation.

Conceptually:

Capability {
    identity
    permissions
    scope
    lifetime
}

A capability MUST be non-forgeable within the applicable security model.

Capabilities SHOULD be used instead of exposing unrestricted native resource access.


---

42. References

ULABI distinguishes:

Reference
Handle
Pointer
Capability
Offset

These concepts MUST NOT be treated as interchangeable.

A Reference represents an ABI-managed relationship.

A Pointer represents an address.

A Handle represents an opaque resource identity.

A Capability represents authority.

An Offset represents a relative location.


---

43. Raw Pointers

Raw pointers MUST NOT be universal ULABI data types.

A raw pointer MAY be used only where the applicable ABI profile guarantees:

shared address space;

valid lifetime;

ownership;

bounds;

alignment;

mutability;

security;

synchronization.


Portable interfaces SHOULD use safer abstractions.


---

44. Buffer Descriptors

A portable buffer SHOULD be described by a descriptor containing:

BufferDescriptor {
    data_reference
    length
    capacity
    element_size
    alignment
    mutability
    ownership
    lifetime
}

Additional metadata MAY include:

stride
shape
encoding
memory_domain
device
access_mode


---

45. Memory Domain

A buffer MAY identify its memory domain.

Examples:

HostMemory
SharedMemory
DeviceMemory
GPUMemory
NPUMemory
FPGARegion
PersistentMemory
RemoteMemory

A memory-domain declaration MUST NOT imply that the receiver can directly access that memory.

Access authority MUST be explicit.


---

46. Tensors

Tensor support SHOULD be provided through a profile rather than the minimal Core.

Conceptually:

Tensor<T> {
    element_type
    rank
    shape
    strides
    layout
    data
}

The Tensor profile SHOULD support:

CPU;

GPU;

NPU;

accelerator;

distributed memory.



---

47. Matrix

Matrix types MAY be represented as specialized tensors.

Conceptually:

Matrix<T> {
    rows
    columns
    layout
    data
}

The representation MUST explicitly define row-major, column-major, or other layout semantics.


---

48. Decimal

Decimal arithmetic SHOULD be defined through an extension profile.

The profile MUST specify:

precision;

scale;

rounding;

overflow;

underflow;

canonical encoding.


Decimal values MUST NOT be silently converted to binary floating-point when exact decimal semantics are required.


---

49. Big Integer

ULABI MAY provide:

BigInt

The representation MUST define:

sign;

magnitude;

byte ordering;

maximum size;

encoding;

overflow/resource limits.


Unbounded mathematical concepts MUST still have explicit implementation resource limits.


---

50. Timestamp

A Timestamp SHOULD be defined by a dedicated time profile.

Conceptually:

Timestamp {
    epoch
    seconds
    fractional_units
}

The representation MUST explicitly define:

epoch;

precision;

timezone semantics;

range.



---

51. Duration

A Duration represents elapsed time.

Conceptually:

Duration {
    seconds
    fractional_units
}

Duration MUST NOT automatically be interpreted as a calendar interval.


---

52. Units and Quantities

ULABI MAY provide a Units profile.

A quantity could contain:

Quantity {
    value
    unit
}

For example:

5 meters
10 kilograms
20 seconds

Unit identity MUST be explicit.


---

53. Streams

A Stream represents potentially unbounded or incrementally produced data.

Conceptually:

Stream<T>

A Stream MUST define:

element type;

ordering;

completion;

cancellation;

failure;

backpressure;

ownership;

lifetime.


Stream semantics belong primarily to the streaming profile.


---

54. Futures

ULABI MAY provide:

Future<T>

A Future represents a value that may become available later.

It MUST define:

Pending
Ready
Failed
Cancelled

Future semantics belong primarily to the asynchronous execution profile.


---

55. Async Functions

An asynchronous function MAY return:

Future<T>

or:

Stream<T>

The calling-convention specification MUST define how these values cross the ABI boundary.


---

56. Recursive Types

ULABI MAY support recursive type definitions.

Example:

Node {
    value: I64
    next: Option<Handle<Node>>
}

Recursive types MUST have explicit lifecycle semantics.

The specification MUST prevent unbounded decoding or allocation attacks.


---

57. Generic Types

ULABI MAY represent generic types through instantiated type descriptors.

For example:

List<I32>

and:

List<String>

MUST have distinguishable type identities when necessary.

ULABI MUST NOT require participating languages to implement generics internally.


---

58. Type Parameters

A generic ULABI contract MAY conceptually contain:

GenericType {
    name
    parameters
    constraints
}

However, binary interoperability MUST ultimately resolve generic parameters into concrete ABI semantics.


---

59. Type Aliases

A type alias MAY provide another name for an existing semantic type.

Example:

UserID = U128

An alias MUST NOT silently create a new semantic type unless explicitly declared as a distinct type.


---

60. Newtype / Distinct Semantic Types

ULABI SHOULD support semantically distinct types sharing the same representation.

Example:

UserID
AccountID
TransactionID

All could use:

U128

internally while remaining semantically distinct.

This prevents accidental interchange of values that happen to share representation.


---

61. Type Constraints

A type MAY define constraints.

Examples:

String(max_length=1024)

U32(range=0..100)

Bytes(max_length=4096)

Constraints MUST be machine-readable where they affect validation.


---

62. Nullability

ULABI MUST distinguish:

Absent
Present
Invalid

where the contract requires these states.

Null MUST NOT be used ambiguously to represent multiple semantic conditions.

Option<T> SHOULD be preferred for optional values.


---

63. Invalid Values

Every ABI-visible type SHOULD define its valid value domain.

An implementation MUST reject values outside that domain unless an explicit recovery policy exists.

Invalid values MUST NOT be silently converted into valid values without declaration.


---

64. Default Values

A ULABI type MAY define a default value.

The absence of a default MUST be distinguishable from:

zero
false
empty
None
null


---

65. Equality

Every semantic type SHOULD define equality semantics.

Equality MAY be:

Bitwise
Structural
Semantic
Identity-based

The contract MUST specify which form applies.


---

66. Ordering

Types that support ordering MUST define the ordering model.

Examples:

Numeric ascending
Lexicographic
Timestamp ordering
Explicit application ordering

An implementation MUST NOT assume an ordering that the ULABI contract does not provide.


---

67. Hashing

ULABI MUST distinguish:

Semantic equality
Hash representation
Implementation hash

A language-specific hash function MUST NOT automatically become a ULABI semantic hash.

Canonical hashing MAY be defined by a future profile.


---

68. Serialization

Serialization converts a ULABI semantic value into a transportable representation.

The serialization system MUST define:

type identity;

encoding;

length;

byte order;

validation;

version;

compatibility;

limits.



---

69. Canonical Encoding

Where canonical encoding is required:

Value
  |
  v
Canonical Encoder
  |
  v
Canonical Bytes

Two implementations encoding the same semantic value MUST produce equivalent canonical representations.

Canonical encoding is especially important for:

hashing;

signatures;

persistence;

distributed communication;

deterministic testing.



---

70. Deserialization

A decoder MUST:

1. Validate the encoding.


2. Validate type identity.


3. Validate version.


4. Validate length.


5. Validate constraints.


6. Validate resource limits.


7. Construct the semantic value.


8. Report failure explicitly.



A decoder MUST NOT blindly trust incoming data.


---

71. Maximum Size

Every implementation MUST be able to impose resource limits.

Possible limits include:

maximum string size
maximum byte sequence size
maximum array length
maximum record nesting
maximum recursion depth
maximum tensor size
maximum message size

An implementation MUST fail safely when limits are exceeded.


---

72. Resource Exhaustion Protection

Type decoding MUST protect against:

allocation bombs;

recursive structures;

oversized collections;

malicious lengths;

integer overflow;

excessive nesting;

decompression bombs;

denial-of-service inputs.


Resource limits MUST be explicit.


---

73. Endianness

Portable canonical representations SHOULD use a defined byte order.

Native ABI mappings MAY use native byte order where explicitly defined.

A receiver MUST know which byte ordering applies.


---

74. Alignment

Fixed-layout ABI-visible types MUST define alignment requirements where alignment affects interpretation or access.

Portable serialized forms SHOULD avoid architecture-dependent alignment assumptions.


---

75. Padding

Padding bytes in ABI-visible structures MUST have defined behavior.

Uninitialized padding MUST NOT leak sensitive information.

Canonical serialization SHOULD avoid unspecified padding.


---

76. Type Conversion

ULABI distinguishes:

Implicit Conversion
Explicit Conversion
Lossless Conversion
Lossy Conversion
Incompatible Conversion

Conversions MUST be explicitly classified.


---

77. Lossless Conversion

A conversion is lossless when the destination type preserves the complete semantic value.

Example:

U8 -> U16

may be lossless.


---

78. Lossy Conversion

A conversion is lossy when information may be removed.

Example:

U64 -> U32

The operation MUST NOT silently discard information unless the contract explicitly permits it.


---

79. Numeric Conversion

Numeric conversions MUST define:

range;

overflow;

rounding;

saturation;

truncation;

NaN;

infinity;

signedness.



---

80. Text Conversion

Text conversion MUST define:

source encoding;

destination encoding;

invalid sequences;

normalization;

replacement policy.



---

81. Structural Compatibility

Two structures MAY be compatible if their externally visible semantic contracts are compatible even when their internal layouts differ.

Compatibility analysis MUST consider:

field identity;

field type;

optionality;

constraints;

defaults;

version.



---

82. Binary Compatibility

Binary compatibility exists when two implementations can exchange ABI representations without reinterpretation errors.

Binary compatibility MUST NOT be inferred merely because source-language types have similar names.


---

83. Semantic Compatibility

Semantic compatibility means that exchanged values retain their intended meaning.

Example:

meters

and:

feet

may use the same numeric representation while remaining semantically different.


---

84. Versioning

Every externally visible evolving type SHOULD have a version.

Conceptually:

TypeName/v1
TypeName/v2

Compatible additions MAY use the same major compatibility family.

Breaking changes MUST create a new compatibility boundary.


---

85. Backward Compatibility

A newer implementation SHOULD be able to consume older compatible representations.

This requires:

stable field identities;

explicit defaults;

unknown-field handling;

version negotiation.



---

86. Forward Compatibility

An older implementation MAY encounter newer fields or variants.

The contract MUST define whether unknown values are:

Ignored
Preserved
Rejected
Converted
Escalated


---

87. Unknown Fields

For extensible records, unknown fields SHOULD be safely ignorable when they do not affect required semantics.

An implementation MUST NOT interpret unknown fields as known fields merely because their physical layout resembles a known field.


---

88. Unknown Variants

Unknown variants MUST have explicit behavior.

Possible behavior:

Reject
Return UnknownVariant
Preserve OpaqueValue
Fallback

The selected behavior MUST be defined by the type contract.


---

89. Type Safety Boundary

A ULABI implementation MUST validate externally supplied type information before using it.

The boundary is:

Untrusted Input
       |
       v
Validation
       |
       v
Type Resolution
       |
       v
Constraint Validation
       |
       v
Safe Value


---

90. Security Requirements

Type handling MUST defend against:

integer overflow;

integer underflow;

truncation;

invalid encodings;

malformed lengths;

recursive structures;

excessive allocations;

type confusion;

use-after-release;

invalid handles;

capability escalation;

parser differentials.



---

91. Type Confusion

A receiver MUST NOT interpret a value as a different type merely because its binary representation happens to be compatible.

For example:

UserID

MUST NOT automatically be accepted as:

AccountID

merely because both use U128.


---

92. Ownership Metadata

A type MAY include ownership metadata:

Owned
Borrowed
Shared
Immutable
Transferred

The memory model defines the detailed ownership contract.

This document defines the requirement that type metadata MUST be capable of expressing ownership where required.


---

93. Lifetime Metadata

A type that crosses an ABI boundary MUST have a defined lifetime model where the value is not self-contained.

Possible states include:

CallScoped
Borrowed
Owned
ReferenceCounted
ExplicitRelease
CapabilityScoped
SessionScoped


---

94. Mutability

A value MAY be:

Immutable
Mutable
ConditionallyMutable
ReadOnly
WriteOnly

Mutability MUST be explicit where it affects ABI behavior.


---

95. Zero-Copy Types

ULABI MAY support zero-copy data exchange.

Zero-copy MUST NOT bypass:

ownership;

lifetime;

bounds;

alignment;

security;

synchronization.


Zero-copy is an optimization, not a semantic requirement.


---

96. Copyable Types

A type MAY be marked as safely copyable.

Copy semantics MUST be explicit.

For resources, copying a handle MUST NOT automatically duplicate the underlying resource unless the resource contract specifies duplication semantics.


---

97. Move Semantics

A type MAY support transfer/move semantics.

A moved value MUST have explicitly defined post-transfer behavior.

For example:

Owned Value
     |
     | transfer
     v
New Owner

The old owner MUST NOT continue to use the value when the contract prohibits it.


---

98. Shared Values

Shared values MAY be represented through:

immutable references;

reference-counted handles;

shared-memory descriptors;

capabilities.


Synchronization semantics MUST be explicitly defined.


---

99. Thread Safety

A type MAY declare:

ThreadSafe
ThreadConfined
Sendable
NonSendable
Synchronised
Immutable

The declaration MUST have defined semantics.


---

100. Distributed Types

A type intended to cross machines MUST define:

serialization;

identity;

lifetime;

failure;

version;

transport independence;

security requirements.


A local pointer MUST NOT become a distributed reference automatically.


---

101. Location-Aware References

A distributed reference MAY contain:

ResourceID
Location
Capability
Version
Expiry

Location transparency MUST NOT hide remote execution semantics.


---

102. Device Types

Device-backed values SHOULD use opaque handles or capabilities.

Examples:

Handle<GPU>
Handle<NPU>
Handle<FPGA>
Handle<Device>

Device-specific memory MUST be explicitly identified.


---

103. Hardware-Aware Numeric Types

Hardware-specific types MAY be introduced through profiles.

Examples:

Vector<T>
SIMD<T,N>
Tensor<T>
DeviceBuffer<T>

The Core MUST remain independent of any particular accelerator.


---

104. Quantum and Future Compute Types

Future compute models MAY define additional types.

Examples include:

QuantumState
QuantumRegister
QuantumMeasurement
FutureComputeObject

Such types MUST be introduced through extensions or profiles.

The Core MUST NOT depend on any particular future hardware model.


---

105. Extension Types

ULABI SHALL support extension types.

An extension type MUST have:

TypeID
Version
Namespace
Encoding
Validation Rules
Compatibility Rules
Security Rules

An extension MUST NOT redefine an existing Core type's meaning.


---

106. Type Namespaces

Type identifiers SHOULD use namespaces.

Example:

org.example.types/Person

Namespaces prevent accidental collisions between independently developed ecosystems.


---

107. Reserved Namespaces

ULABI SHOULD reserve namespaces for:

ulabi.core
ulabi.std
ulabi.profile
ulabi.extension

The governance specification will define allocation rules.


---

108. Type Registry

ULABI SHOULD eventually provide a registry for standardized public type identifiers.

The registry MAY contain:

TypeID
Owner
Version
Definition
Encoding
Compatibility
Status

The registry MUST NOT become a mandatory centralized runtime dependency.


---

109. Offline Operation

Type resolution MUST be possible without permanent network access.

A conformant implementation SHOULD be capable of operating from:

local metadata;

embedded schemas;

cached registry data;

signed packages.



---

110. Deterministic Type Resolution

Given the same valid type identifier and version, an implementation SHOULD resolve the same semantic definition.

Ambiguous type resolution MUST be rejected.


---

111. Schema

A ULABI schema describes the structure and constraints of a type.

A schema SHOULD include:

SchemaID
Version
TypeDefinitions
Constraints
Encoding
CompatibilityRules

Schemas MUST themselves be versioned.


---

112. Schema Validation

Schema validation MUST occur before unsafe use of externally supplied values.

A validator SHOULD check:

Type
Version
Length
Structure
Constraints
Ownership
Capabilities
Resource Limits


---

113. Schema Evolution

Schemas SHOULD support additive evolution.

Preferred evolution:

Add optional field
Add extension
Add compatible variant
Add metadata

Dangerous evolution includes:

Change field meaning
Change required field type
Change numeric semantics
Remove required field
Change encoding

Breaking changes MUST be versioned appropriately.


---

114. Canonical Type Descriptor

ULABI SHOULD eventually define a canonical machine-readable descriptor.

Conceptually:

TypeDescriptor {
    id
    version
    kind
    representation
    constraints
    ownership
    lifetime
    mutability
    encoding
}

The exact representation SHALL be specified separately.


---

115. Type Fingerprints

ULABI MAY define cryptographic type fingerprints.

Conceptually:

TypeFingerprint =
    Hash(CanonicalTypeDescriptor)

Fingerprints MAY be used for:

compatibility checking;

caching;

validation;

distributed negotiation;

reproducibility.



---

116. Type Compatibility Classes

ULABI SHOULD classify compatibility as:

IDENTICAL
COMPATIBLE
CONVERTIBLE
LOSSY
INCOMPATIBLE
UNKNOWN

Tools SHOULD expose these classifications.


---

117. ABI Difference Detection

ULABI tooling SHOULD be capable of comparing two type definitions.

Example:

Type v1
    |
    v
Type v2
    |
    v
Compatibility Analyzer
    |
    +-- Compatible
    +-- Conditionally Compatible
    +-- Breaking

The analyzer SHOULD identify:

field changes;

type changes;

encoding changes;

alignment changes;

ownership changes;

lifetime changes;

constraint changes;

semantic changes.



---

118. Type-Level Feature Negotiation

Implementations MAY negotiate supported types.

Example:

Provider:
    supports String
    supports Bytes
    supports Tensor

Consumer:
    supports String
    supports Bytes

Negotiation:
    String
    Bytes

Unsupported types MUST result in explicit negotiation failure or conversion.


---

119. Graceful Degradation

A type extension MAY define fallback behavior.

Example:

Tensor
   |
   +-- GPU representation
   |
   +-- CPU representation
   |
   +-- Serialized representation

Fallback MUST preserve semantics.

Performance degradation MUST NOT silently become semantic degradation.


---

120. Type Marshalling

A ULABI adapter is responsible for converting language-native values into ULABI boundary values.

Conceptually:

Native Value
     |
     v
Marshaller
     |
     v
ULABI Value
     |
     v
Boundary

The adapter MUST preserve semantic meaning.


---

121. Unmarshalling

The reverse process is:

Boundary Value
     |
     v
Validator
     |
     v
Unmarshaller
     |
     v
Native Value

The destination language MAY choose its own internal representation.


---

122. Marshalling Failure

Marshalling MUST fail explicitly if:

the value cannot be represented;

a constraint is violated;

ownership cannot be transferred;

lifetime cannot be guaranteed;

required capabilities are absent;

conversion would be unsafe.



---

123. No Silent Semantic Conversion

ULABI MUST NOT silently perform semantic transformations.

For example:

meters -> feet
UTC -> local time
USD -> EUR
UTF-8 -> lossy text

must be explicit conversions.


---

124. Type Metadata and Effects

Type contracts MAY interact with function effects.

For example:

Function:
    read_buffer(Buffer)

may declare:

ReadsMemory

while:

Function:
    mutate_buffer(Buffer)

may declare:

WritesMemory

The effect system is specified separately.


---

125. Type Metadata and Security

Type descriptors MAY declare security requirements.

Examples:

RequiresCapability(DeviceAccess)
RequiresEncryption
Sensitive
Confidential
IntegrityProtected
NonExportable

Security semantics belong to the security profiles.


---

126. Sensitive Types

A type MAY be marked sensitive.

Examples:

SecretKey
Credential
AuthenticationToken
PrivateData

Sensitive values SHOULD NOT appear in:

ordinary logs;

debugging output;

crash reports;

telemetry;

unprotected traces.



---

127. Secret Memory

Sensitive types SHOULD use memory protections appropriate to the platform.

Possible protections include:

restricted access;

explicit zeroization;

non-swappable memory;

hardware-backed storage.


These belong to applicable security profiles.


---

128. Deterministic Representation

For types requiring canonical representation:

Same Semantic Value
        |
        v
Same Canonical Representation

This is required for reproducibility and cryptographic operations.


---

129. Cryptographic Representation

When a type participates in signatures or hashes, its canonical representation MUST be unambiguous.

No implementation-specific memory layout may be hashed as a universal representation unless explicitly specified.


---

130. Type Serialization Security

Serialization MUST defend against:

duplicate fields;

ambiguous encodings;

integer overflow;

invalid lengths;

invalid discriminants;

recursive bombs;

type confusion.



---

131. Parser Differential Protection

Different implementations MUST interpret canonical ULABI representations consistently.

The conformance suite SHOULD include differential tests across independent implementations.


---

132. Fuzz Testing

Every Core type parser SHOULD eventually have fuzz tests covering:

valid inputs;

invalid inputs;

boundary values;

maximum values;

malformed encodings;

random data;

nested structures.



---

133. Property Testing

Type implementations SHOULD use property-based tests.

Examples:

encode(decode(x)) == canonical(x)

and:

decode(encode(x)) == x

where applicable.


---

134. Round-Trip Requirement

For types with reversible canonical encoding:

Value
  |
Encode
  |
Bytes
  |
Decode
  |
Value'

The semantic value of Value' MUST equal the semantic value of Value.


---

135. Cross-Language Conformance

The conformance suite MUST eventually test:

Language A
    |
    v
ULABI
    |
    v
Language B

and:

Language B
    |
    v
ULABI
    |
    v
Language A

The same semantic result MUST be obtained in both directions where the contract requires symmetry.


---

136. Reference Test Vectors

Every Core type SHOULD eventually have canonical test vectors.

Example:

Type:
    U32

Value:
    42

Canonical Encoding:
    <defined by encoding specification>

Test vectors MUST be stable across implementations.


---

137. Conformance Levels

Type support SHOULD eventually be divided into levels.

Level 0 — Metadata

Implementation can identify ULABI types.

Level 1 — Core Scalars

Implementation supports:

Bool
I8..I128
U8..U128
F32
F64
Byte
Char
Unit

Level 2 — Core Structures

Implementation supports:

Array
Record
Enum
Variant
Option
Result

Level 3 — Resource Types

Implementation supports:

Handle
Capability
Buffer
Resource

Level 4 — Advanced Types

Implementation supports applicable profiles:

Stream
Future
Tensor
Timestamp
Duration
Decimal
BigInt


---

138. Mandatory Core Types

The initial mandatory Core type set SHOULD be:

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

Byte
Char
Unit

String
Bytes

Array
Record
Enum
Variant
Option
Result

Handle

The final normative Core set SHALL be frozen before ULABI 1.0.


---

139. Optional Profiles

The following types SHOULD be profile-based:

Map
Set
Tuple
BigInt
Decimal
Timestamp
Duration
Quantity
Stream
Future
Tensor
Matrix
DeviceBuffer
DistributedReference
QuantumState


---

140. Implementation Rule

An implementation MUST NOT claim full ULABI type conformance merely because it can map a subset of types.

It MUST declare its supported type/profile set.

Example:

ULABI Core:
    Supported

ULABI Collections:
    Partial

ULABI Async:
    Supported

ULABI Tensor:
    Unsupported


---

141. Compatibility Declaration

Implementations SHOULD expose machine-readable compatibility metadata.

Conceptually:

TypeCapabilities {
    supported_types
    supported_versions
    supported_profiles
    conversion_rules
    limits
}


---

142. Resource Limits

Every implementation MUST define operational limits.

Examples:

max_string_bytes
max_bytes
max_array_length
max_record_depth
max_variant_depth
max_tensor_elements
max_type_nesting

An implementation MAY choose stricter limits than the protocol maximum.


---

143. Failure Semantics

Type operations MUST use explicit failure semantics.

Examples:

InvalidEncoding
InvalidType
UnsupportedType
UnsupportedVersion
ConstraintViolation
Overflow
Underflow
OutOfBounds
ResourceLimitExceeded
OwnershipViolation
LifetimeViolation
CapabilityDenied


---

144. No Undefined ABI Behavior

ULABI MUST avoid undefined behavior at the interoperability boundary.

If behavior cannot be universally defined, it MUST be:

explicitly constrained;

moved into a profile;

marked implementation-defined;

or rejected.



---

145. Implementation-Defined Behavior

Implementation-defined behavior MAY exist where necessary.

However, it MUST be:

1. Documented.


2. Machine-readable where possible.


3. Discoverable.


4. Versioned.


5. Testable.




---

146. Profile Interaction

A type MAY participate in multiple profiles.

For example:

Buffer
   |
   +-- Memory Profile
   +-- Security Profile
   +-- Streaming Profile
   +-- Distributed Profile
   +-- Hardware Profile

The intersection of profile requirements MUST be explicit.


---

147. Self-Healing Interaction

The ULABI self-healing profile MAY use type metadata to diagnose failures.

For example:

Type violation
      |
      v
Detect
      |
      v
Diagnose
      |
      v
Known recovery policy?
   /           \
 YES            NO
 |              |
Recover        Escalate
 |
Verify
 |
Healthy?
 /     \
YES     NO
 |       |
Done   Rollback

Self-healing MUST NOT arbitrarily modify a type definition or ABI contract.

Recovery MUST operate within explicitly authorized policies.


---

148. Type Contract Immutability

Once a ULABI type version is published as stable, its semantic meaning MUST NOT silently change.

A breaking semantic change requires a new version or type identity.


---

149. Governance

Changes to standardized Core types MUST be governed through the ULABI governance process.

No individual implementation may unilaterally redefine:

Bool
I32
String
Bytes
Option
Result
Handle

or any other standardized Core type.


---

150. Version Status

Types SHOULD have lifecycle states:

Experimental
Draft
Stable
Deprecated
Retired

A retired type MUST remain documented for historical compatibility where required.


---

151. Deprecation

Deprecation MUST provide:

replacement type;

migration guidance;

compatibility period;

conformance status.


A deprecated type SHOULD NOT be removed immediately from implementations.


---

152. Formal Verification

Critical type implementations SHOULD be suitable for formal verification.

Particularly important areas include:

integer encoding;

length validation;

bounds checking;

canonical encoding;

type discrimination;

ownership metadata;

resource limits.



---

153. Reference Implementation

ULABI SHOULD eventually provide a small reference implementation of the type system.

The reference implementation SHOULD prioritize:

correctness;

clarity;

determinism;

testability;

portability.


It SHOULD NOT be treated as the definition of ULABI.

The specification remains authoritative.


---

154. Multiple Implementations

ULABI MUST encourage multiple independent implementations.

At minimum, the conformance ecosystem SHOULD eventually include implementations written in more than one programming language.

This prevents accidental coupling between:

ULABI Specification

and:

One Implementation


---

155. Integration With Core ABI

docs/abi/core-abi.md depends on this document for:

primitive types;

composite types;

handles;

boundary representations;

type identity;

compatibility semantics.


The Core ABI document defines how these types participate in ABI calls.

This document defines what those types mean.


---

156. Integration With Calling Convention

docs/abi/calling-convention.md SHALL define how types are physically passed.

For example:

ULABI Type
    |
    v
Logical Argument
    |
    v
Calling Convention
    |
    +-- Register
    +-- Stack
    +-- Reference
    +-- Descriptor

This document MUST NOT define architecture-specific register allocation.


---

157. Integration With Memory Model

docs/abi/memory-model.md SHALL define:

ownership;

borrowing;

allocation;

lifetime;

sharing;

release;

pointer safety;

zero-copy.


This document defines the type metadata that the memory model operates upon.


---

158. Integration With Stack Model

docs/abi/stack-model.md SHALL define how stack-resident ABI values are represented.

This document supplies the type size, alignment, and layout semantics required by that document.


---

159. Integration With Register Model

docs/abi/register-model.md SHALL define how logical ULABI values map to machine registers.

This document defines the semantic types.

The register model defines physical representation.


---

160. Integration With Return Values

docs/abi/return-values.md SHALL define how:

Result<T,E>
Option<T>
Unit

and other return types cross function boundaries.


---

161. Integration With Error Model

docs/abi/exception-model.md SHALL define how ULABI errors are transported.

This document provides the semantic Error and Result types.


---

162. Integration With Interoperability

docs/interoperability/language-interoperability.md SHALL define how language-native types map into ULABI types.

The mapping MUST be explicit.

Example:

Rust String
     |
     v
ULABI String
     |
     v
Python str

or:

C struct
     |
     v
ULABI Record
     |
     v
Zamani record

No language is privileged.


---

163. Integration With FFI

docs/interoperability/foreign-function-interface.md SHALL define how foreign functions expose ULABI-compatible types.

The FFI layer MUST use the type contracts defined here.


---

164. Integration With Serialization

docs/distributed/serialization.md SHALL use these type definitions when values cross process or machine boundaries.

Serialization MUST NOT redefine semantic types.


---

165. Integration With Security

docs/security/security-model.md SHALL use type metadata for:

sensitive values;

capabilities;

handles;

resource boundaries;

validation;

authority.



---

166. Integration With Self-Healing

docs/reliability/self-healing.md SHALL treat type violations as observable failures.

It MUST NOT automatically modify type contracts.

Recovery MAY include:

retry;

fallback representation;

alternate implementation;

rollback;

escalation.



---

167. Integration With Compatibility

docs/compatibility/feature-negotiation.md SHALL use type capabilities to negotiate supported types.

Example:

Provider:
    String
    Bytes
    Tensor

Consumer:
    String
    Bytes

Intersection:
    String
    Bytes


---

168. Integration With Conformance

docs/standards/conformance.md SHALL define how implementations prove support for these types.

Each type SHOULD have:

positive tests;

negative tests;

boundary tests;

encoding tests;

compatibility tests;

fuzz tests.



---

169. Integration With Test Suite

docs/standards/test-suite.md SHALL provide test vectors for every standardized type.

The test suite MUST test independent implementations against the same vectors.


---

170. Integration With Certification

Certification SHOULD verify:

Type Identity
Type Encoding
Type Validation
Type Compatibility
Type Limits
Type Security
Type Evolution

Certification MUST test actual behavior rather than documentation claims.


---

171. File Ownership

The following files own different concerns:

ULABI-DESIGN.md
    Overall architecture

ULABI-SPEC.md
    System-wide normative rules

docs/abi/core-abi.md
    Core ABI contract

docs/abi/data-types.md
    Universal type semantics

docs/abi/calling-convention.md
    Argument/result physical mapping

docs/abi/memory-model.md
    Ownership/lifetime/memory

docs/abi/stack-model.md
    Stack representation

docs/abi/register-model.md
    Register mapping

docs/abi/exception-model.md
    Error/exception transport

docs/abi/return-values.md
    Return-value ABI

docs/type-system/*
    Advanced type-system semantics

docs/interoperability/*
    Language adaptation

docs/distributed/serialization.md
    Distributed representation

docs/security/*
    Security semantics

docs/compatibility/*
    Version and negotiation semantics

docs/standards/*
    Conformance and certification

Each file MUST have a single primary responsibility.


---

172. Implementation Order

The implementation SHOULD proceed in dependency order.

Phase 1 — Independent Foundations

Implement and stabilize:

schemas/type-descriptor
schemas/type-id
schemas/version
schemas/constraints

These should have minimal dependencies.


---

Phase 2 — Core Scalar Types

Implement:

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
Byte
Char
Unit

These should be independently testable.


---

Phase 3 — Core Boundary Types

Implement:

String
Bytes
Array
Record
Enum
Variant
Option
Result

These depend on the scalar type layer.


---

Phase 4 — Identity and Metadata

Implement:

TypeID
TypeDescriptor
SchemaID
Version
TypeFingerprint

These integrate the type system with the ABI identity system.


---

Phase 5 — Resource Types

Implement:

Handle
Capability
BufferDescriptor
Resource

These depend on:

Type metadata
Memory model
Security model


---

Phase 6 — ABI Integration

Integrate with:

core-abi.md
calling-convention.md
return-values.md
exception-model.md
memory-model.md


---

Phase 7 — Language Adapters

Create independent adapters.

For example:

C Adapter
Rust Adapter
Go Adapter
Python Adapter
Java Adapter
Zamani Adapter
Sankofa Adapter

No adapter becomes part of the ULABI specification itself.


---

Phase 8 — Advanced Profiles

Only after the Core is stable:

Async
Streaming
Tensor
Distributed
Hardware
Security
Real-Time
Self-Healing


---

173. Files Required for This Component

The following files are directly involved with the ULABI type system.

Primary specification

docs/abi/data-types.md

This file defines the universal type contract.


---

Supporting schemas

schemas/type-id.schema
schemas/type-descriptor.schema
schemas/type-constraints.schema
schemas/type-capabilities.schema

These define machine-readable metadata.


---

Test vectors

tests/types/

This directory contains:

primitive/
strings/
bytes/
arrays/
records/
enums/
variants/
option/
result/
handles/


---

Conformance tests

conformance/types/

These verify that implementations conform to the type specification.


---

Examples

examples/types/

Examples MUST demonstrate:

primitive values;

structures;

optional values;

results;

handles;

cross-language mappings.



---

174. File Completion Rule

Once docs/abi/data-types.md is approved, later files MUST reference it rather than redefining its type semantics.

For example:

calling-convention.md MUST say how String is passed.

It MUST NOT redefine what a String means.

Similarly:

memory-model.md MUST define ownership of Buffer.

It MUST NOT redefine Buffer itself.

This prevents repeated editing and architectural drift.


---

175. No Circular Specification Dependencies

The dependency direction SHOULD be:

Type Identity
      |
      v
Primitive Types
      |
      v
Composite Types
      |
      v
Type Metadata
      |
      v
Memory / Ownership
      |
      v
Calling Convention
      |
      v
Runtime / Language Adapters
      |
      v
Distributed / Advanced Profiles

Higher layers MUST NOT redefine lower-layer semantics.


---

176. Definition of Done

This document is considered complete for the current design phase when:

all Core types are identified;

semantic meanings are defined;

type identity is defined;

type metadata is defined;

ownership interaction is defined;

lifetime interaction is defined;

mutability is defined;

nullability is defined;

encoding requirements are defined;

compatibility rules are defined;

resource limits are defined;

security requirements are defined;

extension rules are defined;

integration points are documented;

implementation order is documented;

conformance requirements are documented.


Implementation-specific binary encodings MAY remain provisional until the canonical encoding specification is finalized.


---

177. Design Freeze Rule

Before ULABI 1.0:

1. Core type names MUST be frozen.


2. Core semantic meanings MUST be frozen.


3. Core encodings MUST be frozen.


4. Type identity rules MUST be frozen.


5. Compatibility rules MUST be frozen.


6. Security requirements MUST be frozen.


7. Conformance tests MUST be published.


8. Reference test vectors MUST be published.



After ULABI 1.0, breaking changes require a new major version.


---

178. Final Architectural Rule

The ULABI type system exists to create interoperability without forcing uniformity.

The rule is:

> Same semantic contract, not necessarily the same implementation.



Therefore:

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
Future Languages
        |
        v
      ULABI
        |
        v
Universal Semantic Boundary

ULABI does not replace the type systems of these languages.

It provides the common contract through which they can communicate.


---

179. Final Principle

The ultimate objective is:

Any Language
     |
     v
ULABI Adapter
     |
     v
Universal Type Contract
     |
     v
ULABI Adapter
     |
     v
Any Other Language

without requiring:

Language A == Language B

without requiring:

Runtime A == Runtime B

without requiring:

Compiler A == Compiler B

without requiring:

OS A == OS B

without requiring:

CPU A == CPU B

and without making:

Zamani == Sankofa

or either language the foundation of ULABI.

ULABI remains the independent universal interoperability contract.

### File order from here

With this fifth document in place, the clean implementation/documentation order is:

1. `ULABI-DESIGN.md` — architecture
2. `ULABI-SPEC.md` — normative system rules
3. `docs/abi/core-abi.md` — core ABI contract
4. `docs/abi/calling-convention.md` — physical/logical call mapping
5. **`docs/abi/data-types.md` — this document**
6. `docs/abi/memory-model.md`
7. `docs/abi/stack-model.md`
8. `docs/abi/register-model.md`
9. `docs/abi/exception-model.md`
10. `docs/abi/return-values.md`

The important architectural decision is that **Document 5 defines the meaning and contracts of types; later documents consume those definitions rather than redefining them**. That is what lets us work on one file to completion without having to rewrite it whenever another component is added.