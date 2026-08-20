I read the current ULABI-DESIGN.md first. It establishes the language-neutral, layered, minimal-Core + profiles architecture, including semantic types, canonical encoding, memory boundaries, versioning, security, distributed operation, and explicit separation between ULABI, Zamani, and Sankofa.

I also checked docs/abi/data-types.md; it is currently only a placeholder, so Design Document 4 can define this component properly without conflicting with existing implementation.

Below is the complete file for:

docs/abi/data-types.md

ULABI Universal Data Types Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: "docs/abi/data-types.md"
Specification Area: Core ABI / Universal Type System
Status: Architecture / Design Specification
Specification Version: 0.1.0-draft
License: Apache-2.0

---

1. Purpose

This document defines the ULABI Universal Data Type System.

The purpose of the ULABI type system is to provide a language-neutral semantic representation for values crossing a ULABI boundary.

ULABI must allow independently designed programming languages, compilers, runtimes, libraries, operating systems, processors, devices, and distributed systems to exchange data without requiring them to share:

- source syntax;
- programming-language type systems;
- compiler implementations;
- runtime implementations;
- memory-management strategies;
- object models;
- operating systems;
- processor architectures;
- vendor-specific representations.

ULABI defines the meaning and boundary contract of data.

It does not require participating languages to internally represent that data in the same way.

---

2. Relationship to ULABI-DESIGN.md

This document is subordinate to the ULABI master architecture.

The master architecture establishes:

«ULABI is the interoperability contract, not a programming language, compiler, runtime, operating system, processor architecture, or vendor platform.»

The ULABI type system follows the same principle.

The type system therefore separates:

Semantic Meaning
       |
       v
ULABI Boundary Type
       |
       v
Implementation Representation

For example:

Language A String
       |
       v
ULABI String
       |
       v
Language B String

Language A and Language B do not need identical internal string implementations.

They only need to satisfy the ULABI String contract.

---

3. Design Goals

The ULABI Universal Type System MUST prioritize:

1. Language neutrality.
2. Explicit semantics.
3. Deterministic representation.
4. Stable type identity.
5. Cross-platform compatibility.
6. Cross-architecture compatibility.
7. Safe conversion.
8. Explicit ownership.
9. Explicit lifetime.
10. Explicit mutability.
11. Explicit nullability.
12. Explicit error behavior.
13. Version compatibility.
14. Forward compatibility.
15. Backward compatibility.
16. Canonical encoding.
17. Validation.
18. Security.
19. Resource limits.
20. Streaming support.
21. Large-data support.
22. Zero-copy interoperability where safe.
23. Formal verifiability.
24. Extensibility.
25. Long-term stability.

---

4. Non-Goals

The ULABI type system does NOT attempt to:

- replace programming-language type systems;
- force languages to adopt one type system;
- define one universal object model;
- require garbage collection;
- require manual memory management;
- require ownership-based memory management;
- require reference counting;
- require pointers to have a specific representation;
- require all languages to support every ULABI type;
- make incompatible types silently compatible;
- hide lossy conversions;
- define application-specific business types in the Core.

ULABI provides interoperability primitives and semantic contracts.

---

5. Type System Architecture

The type system is divided into layers.

+-------------------------------------------------------+
| Application Semantic Types                            |
+-------------------------------------------------------+
| ULABI Standard Semantic Types                         |
+-------------------------------------------------------+
| ULABI Structural Types                                |
+-------------------------------------------------------+
| ULABI Primitive Types                                 |
+-------------------------------------------------------+
| Canonical Boundary Representation                     |
+-------------------------------------------------------+
| Implementation Representation                         |
+-------------------------------------------------------+

The Core MUST remain small.

More specialized types SHOULD be introduced through standardized extensions and profiles.

---

6. Type Categories

ULABI types are divided into the following categories:

6.1 Primitive Types

Primitive types represent fundamental values.

Initial Core primitives:

- "Bool"
- "Int"
- "UInt"
- "Float"
- "Char"
- "Unit"

---

6.2 Binary Types

Binary types represent raw data.

Core binary type:

- "Bytes"

---

6.3 Text Types

Core textual type:

- "String"

ULABI uses Unicode semantics.

UTF-8 is the preferred canonical interchange representation.

---

6.4 Structural Types

Structural types compose other types.

Examples:

- "List<T>"
- "Record"
- "Tuple"
- "Map<K,V>"
- "Set<T>"

---

6.5 Algebraic Types

ULABI supports semantic algebraic types.

Examples:

- "Enum"
- "Variant"
- "Option<T>"
- "Result<T,E>"

---

6.6 Resource Types

Resource types represent externally managed resources.

Examples:

- "Handle"
- "Resource"
- "Capability"

Resource semantics MUST be explicit.

---

6.7 Temporal Types

Standardized temporal extensions may include:

- "Timestamp"
- "Duration"
- "Date"
- "Time"
- "TimeZone"

Temporal semantics MUST NOT depend on a local machine's clock representation.

---

6.8 Numeric Extensions

Future standardized numeric types may include:

- "Decimal"
- "BigInteger"
- fixed-point numbers;
- arbitrary-precision floating-point values;
- complex numbers;
- rational numbers.

---

6.9 Scientific and Accelerator Types

Optional profiles may define:

- "Tensor"
- "Matrix"
- "Vector"
- "SparseTensor"
- "Complex"
- accelerator buffers;
- device memory references.

These MUST NOT become mandatory Core types unless there is a demonstrated interoperability requirement.

---

7. Universal Type Identity

Every externally visible ULABI type MUST have a stable identity.

A type identity MUST NOT depend solely on:

- source-language name;
- memory address;
- compiler-generated symbol;
- compiler version;
- process ID;
- machine-specific pointer;
- local runtime identifier.

A type identity should be represented through a stable identifier.

Conceptually:

TypeID
{
    namespace
    name
    version
    identity
}

The exact wire representation will be defined by the Core ABI and canonical encoding specifications.

---

8. Type Names

Human-readable type names are descriptive.

They are not sufficient as the sole identity mechanism.

For example:

User

is ambiguous.

A stable identity should distinguish:

example.identity.User

from:

another.identity.User

The specification MUST prevent accidental type collisions.

---

9. Primitive Type: Bool

"Bool" represents exactly two semantic states:

true
false

No third state is permitted.

The canonical representation MUST define:

- encoding;
- decoding;
- validation;
- invalid representations;
- size;
- compatibility behavior.

Implementations MUST NOT interpret arbitrary non-zero values as valid ULABI booleans unless explicitly permitted by a compatibility profile.

---

10. Integer Types

ULABI distinguishes:

Int
UInt

Signed and unsigned integer semantics MUST be explicit.

An implementation MUST NOT assume that:

int

in one language has the same range as:

int

in another language.

---

11. Integer Widths

ULABI may provide explicitly sized integer variants:

Int8
Int16
Int32
Int64
Int128

UInt8
UInt16
UInt32
UInt64
UInt128

Additional widths may be standardized later.

An implementation MUST NOT silently reinterpret one integer width as another when doing so changes the representable range.

---

12. Integer Overflow

Integer overflow behavior MUST be explicit.

Possible policies include:

- trap;
- return error;
- checked operation;
- wrapping operation;
- saturating operation.

The ABI MUST NOT silently select an overflow behavior merely because the source language happens to use that behavior internally.

---

13. Floating-Point Types

ULABI may support standardized floating-point representations including:

Float32
Float64

Future profiles may support additional formats.

Floating-point semantics MUST define:

- precision;
- exponent range;
- NaN;
- positive infinity;
- negative infinity;
- positive zero;
- negative zero;
- rounding;
- comparison;
- canonical encoding.

---

14. NaN Semantics

ULABI implementations MUST explicitly define behavior involving NaN values.

The following MUST NOT be silently assumed:

NaN == NaN

Canonical serialization SHOULD provide deterministic treatment of NaN representations where deterministic encoding is required.

---

15. Character Type

"Char" represents a Unicode scalar value.

It MUST NOT be defined merely as:

one byte

or:

one machine word

The implementation representation may differ.

The ULABI boundary semantics remain language-neutral.

---

16. String Type

"String" represents Unicode text.

ULABI SHOULD use UTF-8 as the canonical boundary encoding.

A ULABI String contract MUST define:

- encoding;
- validity;
- length semantics;
- maximum permitted size;
- malformed sequence behavior;
- normalization policy;
- ownership;
- lifetime;
- mutability.

---

17. String Length

ULABI MUST distinguish between:

- byte length;
- code-point count;
- grapheme count.

The boundary contract MUST specify which measurement is being used.

Implementations MUST NOT assume that:

length(String)

means the same thing in every language.

---

18. String Normalization

ULABI MUST NOT silently normalize text unless the contract explicitly requires it.

Where normalization matters, a profile SHOULD identify the normalization form.

This prevents two systems from silently changing application data.

---

19. Bytes

"Bytes" represents arbitrary binary data.

String != Bytes

A byte sequence MUST NOT automatically be interpreted as text.

"Bytes" may contain:

- zero bytes;
- arbitrary binary values;
- encoded documents;
- cryptographic material;
- compressed data;
- application-defined binary formats.

---

20. Unit

"Unit" represents successful completion where no value is returned.

Example:

Result<Unit, Error>

The Unit value carries no application-defined payload.

---

21. List

ULABI defines an ordered collection abstraction:

List<T>

A List contract MUST define:

- element type;
- ordering;
- element count;
- maximum size;
- encoding;
- ownership;
- lifetime;
- mutation semantics.

Example:

List<Int>

---

22. Tuple

ULABI may define positional structures:

Tuple<T1,T2,...,Tn>

Tuple elements are identified by position.

Example:

Tuple<String, UInt, Bool>

A tuple is distinct from a named Record.

---

23. Record

A Record represents named fields.

Example:

Person {
    name: String
    age: UInt
    active: Bool
}

Each field MUST have:

- stable identity;
- type;
- required/optional status;
- compatibility behavior;
- ownership semantics where applicable.

---

24. Record Evolution

Records MUST support controlled evolution.

For example, version 1:

Person {
    name
    age
}

Version 2:

Person {
    name
    age
    country
}

A compatible implementation SHOULD be able to ignore fields it does not understand when the contract permits it.

Unknown fields MUST NOT be silently discarded when the contract requires preservation.

---

25. Optional Fields

A Record may contain optional fields.

Example:

Person {
    name: String
    age: UInt
    nickname: Option<String>
}

Optionality MUST be distinguishable from a missing or malformed field.

---

26. Enum

An Enum represents a closed set of named values.

Example:

Status {
    Pending
    Active
    Complete
    Failed
}

Enum members MUST have stable identifiers.

Numeric positions MUST NOT be relied upon as stable identity unless explicitly standardized.

---

27. Variant

A Variant represents one of multiple possible alternatives.

Example:

Shape =
    Circle(radius)
    Rectangle(width, height)
    Triangle(a, b, c)

Each variant MUST have a stable identity.

Unknown variants MUST have explicitly defined behavior.

---

28. Option

ULABI defines:

Option<T>

with two semantic states:

None
Some(value)

Option MUST be distinct from:

null

unless an explicit language adapter defines an equivalent mapping.

---

29. Result

ULABI defines:

Result<T,E>

with:

Ok(value)
Err(error)

Result provides a language-neutral error-return mechanism.

It allows languages with different exception and error systems to interoperate without requiring identical error models.

---

30. Error Values

Errors SHOULD themselves be represented using structured ULABI types.

Example:

Error {
    code: ErrorCode
    message: String
    details: Option<Bytes>
}

Errors MUST have stable machine-readable identity.

Human-readable messages MUST NOT be the only error identifier.

---

31. Map

A Map represents key-value associations:

Map<K,V>

The Map specification MUST define:

- key restrictions;
- equality;
- uniqueness;
- ordering;
- duplicate handling;
- canonical encoding;
- maximum size.

If ordering is not guaranteed, implementations MUST NOT assume ordering.

---

32. Set

A Set represents unique values.

Set<T>

The Set specification MUST define:

- equality;
- uniqueness;
- ordering;
- canonical representation.

---

33. Generic Types

ULABI may describe parameterized types:

List<T>
Map<K,V>
Option<T>
Result<T,E>

Generic parameters MUST resolve to concrete boundary types before an actual ABI operation unless the interface explicitly supports runtime type descriptors.

---

34. Type Descriptors

ULABI SHOULD support runtime type descriptors for dynamic interoperability.

A type descriptor may contain:

TypeID
Kind
Version
Parameters
Constraints
Encoding
Semantics

Type descriptors are especially useful for:

- dynamic languages;
- plugin systems;
- RPC;
- distributed systems;
- reflection;
- debugging;
- schema validation;
- dynamic loading.

---

35. Dynamic Values

A future Dynamic Value profile may support:

DynamicValue {
    TypeID
    Version
    Value
}

Dynamic values MUST remain bounded and validated.

Implementations MUST NOT use dynamic typing as a mechanism to bypass security or compatibility validation.

---

36. Type Compatibility

ULABI compatibility MUST distinguish:

Identical
Compatible
Convertible
Conditionally Compatible
Incompatible
Unknown

Two types must not be considered compatible merely because their names match.

---

37. Safe Conversion

Conversions SHOULD be classified as:

Lossless
Potentially Lossy
Lossy
Invalid

Example:

Int32 -> Int64

may be lossless.

Whereas:

Int64 -> Int32

may be lossy.

An implementation MUST NOT silently perform a potentially lossy conversion when the interface requires exact preservation.

---

38. Semantic Types

ULABI should eventually support semantic type annotations.

Examples:

Meters
Seconds
Currency
Temperature
Probability
Percentage
Identifier
EmailAddress
URL
UUID

Semantic types SHOULD be layered above primitive representations.

For example:

Meters = Float64 + UnitMetadata

This allows implementations to preserve semantic meaning without forcing a particular programming-language representation.

---

39. Units

A Units Profile may define dimensional metadata.

Example:

Quantity {
    value: Float64
    unit: UnitID
}

Implementations SHOULD reject incompatible dimensional operations where the profile requires safety.

---

40. Identifiers

ULABI should support standardized identifier types such as:

UUID
URI
ResourceID
InterfaceID
TypeID
FunctionID
CapabilityID

Identifiers MUST have explicit encoding and comparison rules.

---

41. Handles

A "Handle" represents an opaque reference to an externally managed resource.

Examples:

- file;
- socket;
- device;
- process;
- shared-memory region;
- accelerator resource.

Handles MUST NOT expose implementation-specific memory addresses as their universal identity.

---

42. Resource

A "Resource" represents an externally managed entity with lifecycle semantics.

A resource contract SHOULD specify:

Create
Acquire
Use
Transfer
Release
Destroy

Not every resource supports every operation.

---

43. Capability

A "Capability" represents an authority granted to a component.

Capabilities MUST be opaque and non-forgeable where the security profile requires it.

Examples:

FileReadCapability
NetworkCapability
GPUCapability
DeviceCapability

The type system itself does not grant authority.

Security profiles determine capability semantics.

---

44. Ownership Metadata

A ULABI value crossing a boundary MUST have explicit ownership semantics when the value is not trivially copied.

Possible states include:

Borrowed
Owned
Shared
Transferred
ImmutableShared

The exact lifecycle contract is defined by the memory model.

---

45. Mutability

ULABI SHOULD distinguish:

Immutable
Mutable
ReadOnly
WriteOnly
SharedMutable

Mutability MUST NOT be inferred solely from the source-language type.

---

46. Nullability

ULABI MUST distinguish:

Absent
Null
Invalid
Unknown
Value

where these states are semantically relevant.

Languages with a single null concept MUST use an adapter that maps their semantics explicitly.

---

47. Pointers

Raw pointers are NOT Core semantic data types.

A raw machine pointer is:

- process-specific;
- architecture-specific;
- potentially unsafe;
- potentially meaningless outside its address space.

Pointers MAY be used internally by implementations.

Cross-boundary pointer exchange requires an explicit memory or shared-memory profile.

---

48. Zero-Copy Data

ULABI may support zero-copy transfer.

Zero-copy MUST NOT mean:

«Ignore ownership and lifetime.»

A zero-copy value MUST have explicit:

- storage ownership;
- lifetime;
- mutability;
- synchronization;
- access permissions;
- invalidation behavior.

---

49. Shared Memory

Shared-memory values require explicit synchronization semantics.

The contract MUST define:

- ownership;
- visibility;
- synchronization;
- mutation;
- lifetime;
- process boundaries;
- security permissions.

---

50. Streaming Types

Large values SHOULD be representable as streams.

Example:

Stream<Bytes>
Stream<Record>
Stream<T>

Streaming prevents the ABI from requiring an entire dataset to exist in memory simultaneously.

---

51. Large Data

ULABI implementations MUST support explicit resource limits.

Examples:

MaximumStringSize
MaximumBytesSize
MaximumListLength
MaximumRecordDepth
MaximumNestingDepth
MaximumMessageSize

Unlimited values MUST NOT be assumed.

"Universal" does not mean unlimited.

---

52. Recursive Types

ULABI may support recursive structures.

Example:

Tree =
    Empty
    Node(
        value: Int,
        left: Tree,
        right: Tree
    )

Recursive types MUST have bounded decoding and validation behavior.

Implementations MUST defend against malicious recursive structures.

---

53. Cyclic Data

Cyclic object graphs are not automatically part of the Core value model.

A reference/cycle profile MAY define them.

Such a profile MUST specify:

- identity;
- reference tracking;
- ownership;
- lifetime;
- cycle detection;
- serialization behavior.

---

54. Canonical Representation

A ULABI value MUST have a deterministic canonical representation whenever canonical encoding is required.

Canonical encoding is important for:

- hashing;
- signatures;
- caching;
- comparison;
- reproducible builds;
- deterministic testing;
- distributed systems.

Equivalent semantic values SHOULD produce equivalent canonical representations.

---

55. Endianness

ULABI boundary representations MUST NOT depend on the host CPU's native byte order.

The canonical encoding MUST define byte order explicitly.

---

56. Alignment

ULABI semantic types MUST NOT assume native machine alignment.

Alignment requirements belong to the implementation ABI or a specific optimized profile.

This allows ULABI to operate across architectures with different alignment requirements.

---

57. Representation Independence

The following are implementation details unless explicitly exposed by a profile:

- pointer size;
- register size;
- stack layout;
- object header layout;
- garbage collector metadata;
- vtable layout;
- compiler-specific padding;
- language-specific object representation.

ULABI interoperates through its defined boundary representation.

---

58. ABI Type Mapping

A language adapter maps local types into ULABI types.

Example:

Language A
    |
    | Type Mapping
    v
ULABI Type
    |
    | Type Mapping
    v
Language B

The adapter MUST document mappings.

Example:

Language A: string
        ->
ULABI: String
        ->
Language B: text object

---

59. Unsupported Types

A language implementation may contain types with no direct ULABI equivalent.

The implementation MUST then choose one of:

1. Provide an explicit adapter.
2. Represent the value using a standardized generic type.
3. Use an extension profile.
4. Reject the operation.

It MUST NOT silently reinterpret an unsupported type.

---

60. Type Erasure

Type erasure MAY be used internally.

However, type information required by the ULABI contract MUST remain available at the boundary.

---

61. Type Versioning

Types MUST be versionable.

A type evolution SHOULD distinguish:

Compatible Change
Breaking Change
Optional Extension
Semantic Change
Encoding Change

A change to the semantic meaning of an existing type MUST be treated as potentially breaking.

---

62. Forward Compatibility

An implementation should be able to encounter future type information without catastrophic failure.

Where permitted:

Known field -> process
Unknown field -> preserve or ignore according to contract

Unknown information MUST NOT automatically become trusted executable behavior.

---

63. Backward Compatibility

New implementations SHOULD be able to consume older valid representations when compatibility is declared.

Compatibility MUST be determined by the contract rather than by best-effort guessing.

---

64. Type Negotiation

Dynamic environments may negotiate supported types.

Example:

Peer A:
Supports String v1
Supports Bytes v1
Supports Record v1

Peer B:
Supports String v1
Supports Bytes v1
Supports Record v2

The negotiation mechanism should identify a mutually compatible representation.

---

65. Capability Discovery

Type capability discovery may report:

SupportedType
SupportedVersion
SupportedEncoding
MaximumSize
OptionalFeature

Discovery MUST NOT itself grant access to protected resources.

---

66. Validation

Every boundary value SHOULD pass validation appropriate to its type.

Validation may include:

- type identity;
- version;
- encoding;
- size;
- structure;
- range;
- ownership;
- capability requirements;
- semantic constraints.

Invalid values MUST produce explicit failures.

---

67. Security Requirements

Type decoding MUST be treated as an attack surface.

Implementations MUST defend against:

- oversized values;
- malformed encodings;
- integer overflow;
- excessive nesting;
- recursive bombs;
- allocation exhaustion;
- parser differentials;
- ambiguous encodings;
- type confusion;
- forged handles;
- invalid capability references.

---

68. Type Confusion Prevention

A value MUST NOT be interpreted as a different semantic type merely because its raw representation happens to be compatible.

For example:

UserID

must not automatically become:

FileID

merely because both are represented as strings.

Semantic type identity matters.

---

69. Resource Limits

Every implementation SHOULD support configurable limits for:

MaximumTypeDepth
MaximumCollectionLength
MaximumStringSize
MaximumBytesSize
MaximumRecordSize
MaximumVariantSize
MaximumAllocation
MaximumStreamWindow

Limits SHOULD fail closed where required by the security profile.

---

70. Determinism

Where a deterministic profile is enabled:

- canonical encoding MUST be deterministic;
- type identity MUST be deterministic;
- field ordering MUST be deterministic where required;
- numeric encoding MUST be deterministic;
- error identity MUST be deterministic.

---

71. Internationalization

ULABI's semantic type system MUST support international text.

It SHOULD support:

- Unicode;
- locale identifiers;
- language identifiers;
- time zones;
- culturally sensitive formatting through higher-level profiles.

ULABI Core SHOULD NOT impose one human language.

---

72. Localization Separation

Data semantics MUST be separated from presentation.

For example:

Amount = 1000
Currency = USD

is preferable to:

"$1,000"

when machine interoperability is required.

---

73. Time

Time SHOULD use explicit semantic types.

For example:

Timestamp
Duration

must not be represented solely through a language-specific integer without defining:

- epoch;
- precision;
- timezone semantics;
- valid range.

---

74. Decimal and Financial Data

A financial profile SHOULD define exact decimal semantics.

Binary floating point MUST NOT automatically be assumed suitable for exact monetary values.

Example:

Decimal {
    coefficient
    scale
}

The exact representation will be specified by a future numeric profile.

---

75. Tensor and Matrix Data

A future accelerator profile may define:

Tensor<T>

with metadata such as:

shape
element_type
layout
stride
device
memory_location

Such functionality belongs to an extension profile rather than being required from every ULABI implementation.

---

76. Future and Async Values

Asynchronous results may use:

Future<T>

The Future type SHOULD be defined by the Async Profile.

The Core type system must remain usable by synchronous systems.

---

77. Streams

Streaming types SHOULD support:

Open
Read
Pause
Resume
Cancel
Close
Error
End

The streaming lifecycle belongs to the Streaming Profile.

---

78. Type Effects

A type MAY carry metadata describing effects.

Examples:

Pure
ResourceBound
CapabilityBound
NonDeterministic
Sensitive

Effects are metadata, not substitutes for the security model.

---

79. Sensitive Data

A Security Profile MAY identify sensitive types.

Examples:

Secret
Credential
PrivateKey
AuthenticationToken
PersonalData

Sensitive data SHOULD support policies for:

- zeroization;
- restricted logging;
- restricted serialization;
- access control;
- redaction.

---

80. Cryptographic Types

A Cryptographic Profile may define:

Hash
Signature
PublicKey
PrivateKey
Ciphertext
Nonce
KeyIdentifier

Cryptographic types MUST identify algorithms and parameters explicitly where required.

---

81. Type Safety Boundary

The ULABI boundary MUST be treated as a type-safety boundary.

The implementation MUST NOT assume that values received from another component are trustworthy merely because they are encoded correctly.

Validation and authorization remain separate responsibilities.

---

82. Type Mapping Contract

Every language binding SHOULD provide a type mapping table.

Example:

ULABI Type       Local Type
--------------------------------
Bool             boolean
Int32            language-specific 32-bit integer
UInt64           language-specific 64-bit integer
String           native text type
Bytes            byte array
List<T>          native collection
Option<T>        optional type
Result<T,E>      result/error type
Record           struct/class/object

Mappings must document:

- lossless conversions;
- lossy conversions;
- ownership;
- lifetime;
- mutability;
- exceptions/errors;
- unsupported cases.

---

83. Type Adapter Safety

Adapters MUST NOT silently:

- truncate integers;
- alter Unicode text;
- discard fields;
- change units;
- change time zones;
- discard precision;
- change ownership;
- change mutability;
- grant capabilities.

Any such behavior must be explicit.

---

84. Cross-Language Equality

ULABI SHOULD distinguish:

Representation Equality
Semantic Equality
Identity Equality

Two values can have different internal representations while being semantically equal.

---

85. Hashing

If a type supports canonical hashing, the hash MUST be based on the canonical semantic representation.

Implementation-specific memory layouts MUST NOT be used as universal hashes.

---

86. Serialization

Serialization is the conversion of a ULABI value into a boundary representation.

Serialization MUST be:

- deterministic where required;
- validated;
- version-aware;
- bounded;
- type-aware.

---

87. Deserialization

Deserialization MUST validate input before exposing the resulting value to application code.

Implementations SHOULD use staged processing:

Input
  |
Decode
  |
Validate
  |
Type Check
  |
Resource Check
  |
Ownership Assignment
  |
Application Value

---

88. Failure Behavior

Type failures MUST produce explicit failure categories.

Possible categories:

InvalidEncoding
UnknownType
UnsupportedVersion
TypeMismatch
RangeError
SizeLimitExceeded
MalformedValue
InvalidReference
CapabilityViolation
ResourceExhausted

Exact error identifiers belong to the Core Error Model.

---

89. Formal Invariants

A conforming ULABI type implementation MUST preserve:

Invariant 1

A valid value has a valid type identity.

Invariant 2

A value cannot be interpreted as another semantic type without an explicit conversion.

Invariant 3

Canonical representations are deterministic when canonical mode is required.

Invariant 4

Invalid representations MUST NOT be exposed as valid application values.

Invariant 5

Ownership and lifetime MUST remain consistent across the boundary.

Invariant 6

Lossy conversions MUST be explicit.

Invariant 7

Resource limits MUST be enforceable.

Invariant 8

Unknown future information MUST NOT automatically grant authority.

Invariant 9

Type evolution MUST preserve declared compatibility guarantees.

Invariant 10

Language-specific implementation details MUST NOT become accidental ULABI requirements.

---

90. Conformance Levels

ULABI may define:

Level 0 — Core Types

Required:

- Bool
- signed integers
- unsigned integers
- floating point
- Char
- String
- Bytes
- Unit

Level 1 — Structural Types

Adds:

- List
- Tuple
- Record
- Enum
- Variant

Level 2 — Result Types

Adds:

- Option
- Result
- standardized errors

Level 3 — Resource Types

Adds:

- Handle
- Resource
- Capability

Level 4 — Advanced Types

Adds selected profiles:

- Map
- Set
- Timestamp
- Duration
- Decimal
- Tensor
- Stream
- Future

Implementations MUST declare exactly which levels and profiles they support.

---

91. Conformance Testing

The ULABI conformance suite SHOULD test:

- encoding;
- decoding;
- type identity;
- integer boundaries;
- floating-point edge cases;
- Unicode;
- malformed strings;
- collection limits;
- nested structures;
- unknown fields;
- unknown variants;
- optional values;
- result values;
- error values;
- ownership;
- lifetime;
- version compatibility;
- canonical representations;
- invalid conversions;
- resource exhaustion.

---

92. Fuzz Testing

The type system SHOULD be fuzz tested against:

- random values;
- malformed values;
- truncated values;
- oversized values;
- recursive structures;
- deeply nested structures;
- invalid type identifiers;
- invalid versions;
- invalid encodings;
- boundary numeric values.

A conforming implementation SHOULD survive malformed input without memory corruption or uncontrolled resource consumption.

---

93. Differential Testing

Multiple ULABI implementations SHOULD be able to process the same canonical test vectors.

Example:

Implementation A
        |
        +---- Test Vector ----+
        |                     |
Implementation B         Implementation C

Equivalent valid values should produce equivalent canonical results.

---

94. Reference Test Vectors

ULABI SHOULD maintain official test vectors for:

- every Core primitive;
- every structural type;
- every canonical encoding;
- boundary values;
- invalid values;
- compatibility cases.

Test vectors SHOULD be implementation-independent.

---

95. Language Independence

ULABI MUST remain independent of any particular programming language.

The following languages may implement ULABI:

- C
- C++
- Rust
- Go
- Java
- C#
- Python
- Swift
- Kotlin
- Fortran
- Ada
- JavaScript
- TypeScript
- Zig
- D
- Julia
- Lua
- Ruby
- PHP
- and future languages.

No language receives privileged status.

---

96. Zamani and Sankofa Independence

Zamani and Sankofa are separate programming languages.

ULABI MUST NOT merge them into one language.

ULABI MUST NOT define their syntax.

ULABI MUST NOT require them to share implementations.

Either language may independently implement a ULABI adapter.

Conceptually:

Zamani
   |
   v
ULABI Adapter
   |
   v
ULABI
   ^
   |
ULABI Adapter
   ^
   |
Sankofa

This is an interoperability relationship, not a language relationship.

The same architecture applies to every other programming language.

---

97. ABI Boundary Principle

The central rule of this specification is:

«A ULABI type describes what a value means at an interoperability boundary, not how a programming language must represent that value internally.»

This principle is mandatory for long-term universality.

---

98. Extensibility

New types MUST be introduced through a controlled extension process.

An extension SHOULD define:

1. Type identity.
2. Semantic meaning.
3. Encoding.
4. Validation.
5. Ownership.
6. Lifetime.
7. Compatibility.
8. Security.
9. Resource limits.
10. Conformance tests.
11. Reference implementation.

No extension should modify the meaning of an existing Core type.

---

99. Stability Rules

Core types SHOULD be extremely stable.

Breaking changes SHOULD require:

- new type identity;
- new major specification version;
- compatibility analysis;
- migration documentation;
- conformance updates.

Existing valid programs SHOULD continue functioning across compatible ULABI versions.

---

100. Future-Proofing

ULABI must be designed for technologies that do not yet exist.

The type system should therefore avoid assumptions about:

- future processors;
- future memory architectures;
- future programming languages;
- future accelerators;
- future operating systems;
- future distributed systems;
- future computing models.

The Core defines stable semantics.

Profiles provide specialized capabilities.

---

101. Security Philosophy

The type system follows:

Parse
  ↓
Validate
  ↓
Authenticate if required
  ↓
Authorize if required
  ↓
Convert
  ↓
Expose

A valid type is not automatically an authorized resource.

Type safety and authorization are separate layers.

---

102. Failure-Oriented Type Design

ULABI assumes that type failures will occur.

Therefore every implementation SHOULD be prepared for:

Malformed Input
      ↓
Detection
      ↓
Classification
      ↓
Containment
      ↓
Explicit Failure
      ↓
Recovery / Retry / Escalation

A malformed value MUST NOT cause undefined behavior.

---

103. Self-Healing Interaction

Self-healing is not part of the semantic definition of a type.

However, the Reliability and Self-Healing Profiles MAY use type metadata to determine safe recovery actions.

For example:

Type
 |
 +-- Safe to retry?
 |
 +-- Idempotent?
 |
 +-- Reconstructible?
 |
 +-- Persistent?
 |
 +-- Sensitive?
 |
 +-- Recoverable?

The type system provides metadata.

The self-healing system determines recovery according to explicit policies.

---

104. No Autonomous Semantic Mutation

An implementation MUST NOT change the meaning of an existing ULABI type merely because:

- decoding failed;
- a type appears unfamiliar;
- recovery was attempted;
- another implementation behaves differently.

Semantic changes require specification-controlled evolution.

---

105. Recommended Core Type Set

The initial Core should remain approximately:

Bool

Int8
Int16
Int32
Int64
Int128

UInt8
UInt16
UInt32
UInt64
UInt128

Float32
Float64

Char
String
Bytes
Unit

List
Tuple
Record
Enum
Variant

Option
Result

Additional types should be evaluated individually.

---

106. Recommended Extension Type Set

Potential extensions include:

Map
Set

Timestamp
Duration
Date
Time

Decimal
BigInteger
Complex

UUID
URI

Handle
Resource
Capability

Stream
Future

Tensor
Matrix
Vector

Hash
Signature
PublicKey
PrivateKey

Quantity
Unit

---

107. Implementation Requirements

A ULABI type implementation SHOULD provide:

Type Registry
Type Identity
Type Validator
Encoder
Decoder
Compatibility Checker
Conversion Engine
Resource Limiter
Canonicalizer
Test Vector Runner

The exact architecture remains implementation-specific.

---

108. Type Registry

A Type Registry MAY maintain:

TypeID
Version
Schema
Encoding
Constraints
Compatibility
Profiles

Registries MUST avoid becoming mandatory centralized infrastructure unless a separate governance specification explicitly defines such a requirement.

Distributed and local registries SHOULD be supported.

---

109. Schema Independence

ULABI schemas should be machine-readable.

Possible future schema formats may include:

ULABI Schema
JSON representation
CBOR representation
Binary representation
Human-readable specification format

The choice of schema representation MUST NOT redefine the semantic type itself.

---

110. Implementation Architecture

A typical implementation may look like:

                 Application
                     |
              Language Adapter
                     |
               Type Mapper
                     |
              ULABI Type Layer
              /       |       \
        Validator  Encoder  Decoder
              \       |       /
                Core ABI
                     |
             Transport / Runtime

---

111. Minimal Implementation

A minimal ULABI implementation should be able to support:

Bool
Int
UInt
Float
String
Bytes
Unit
Record
Option
Result

plus:

- type identity;
- validation;
- canonical encoding;
- versioning;
- compatibility checking.

This makes experimentation possible without requiring the entire ULABI ecosystem.

---

112. Reference Implementation

The first reference implementation SHOULD prioritize correctness over maximum performance.

It should provide:

- readable code;
- extensive tests;
- deterministic behavior;
- security validation;
- canonical test vectors;
- compatibility tests;
- fuzz testing.

Optimized implementations can follow.

---

113. Performance

The type system SHOULD support multiple implementation strategies:

Canonical Copy
Zero Copy
Shared Memory
Streaming
Lazy Decode
Incremental Decode

Optimization MUST NOT change semantic behavior.

---

114. Zero-Copy Safety Rule

Zero-copy MUST never override:

- ownership;
- lifetime;
- mutability;
- synchronization;
- authorization;
- memory safety.

If zero-copy cannot satisfy the contract safely, the implementation MUST fall back to copying or reject the operation.

---

115. Distributed Type Semantics

A type valid locally may not automatically be suitable remotely.

Distributed profiles MUST consider:

- serialization;
- versioning;
- network failure;
- partial delivery;
- retries;
- idempotency;
- authentication;
- authorization;
- resource limits.

ULABI MUST NOT hide these differences.

---

116. Remote Type Identity

Remote type identity MUST be stable across:

- process restarts;
- machines;
- operating systems;
- architectures;
- compiler versions.

Local memory addresses MUST NOT be used as remote type identities.

---

117. Embedded Systems

The type system SHOULD support constrained systems.

Implementations MAY disable:

- dynamic type discovery;
- reflection;
- large collections;
- dynamic allocation;
- advanced profiles.

A small embedded implementation can therefore conform to a constrained ULABI profile.

---

118. Safety-Critical Systems

A Safety-Critical Profile SHOULD define stricter requirements for:

- deterministic behavior;
- bounded memory;
- bounded execution;
- validation;
- failure handling;
- formal verification;
- certification evidence.

The Core remains general-purpose.

---

119. Formal Verification

Critical type operations SHOULD be suitable for formal verification.

Especially:

- integer encoding;
- integer decoding;
- bounds checking;
- length validation;
- canonical encoding;
- type identity;
- compatibility checking.

---

120. Compatibility Matrix

Implementations SHOULD expose a compatibility matrix.

Example:

                    Provider
                V1       V2
Consumer V1     ✓        ✓
Consumer V2     ✓        ✓/conditional
Consumer V3     ?        ?

Compatibility MUST be determined using the formal specification.

---

121. Type Contract Template

Every future ULABI type specification SHOULD follow:

Type
Purpose
Semantic Definition
Type Identity
Version
Canonical Representation
Encoding
Decoding
Validation
Constraints
Ownership
Lifetime
Mutability
Error Behavior
Security Requirements
Compatibility
Extension Points
Conformance Tests
Reference Implementation

This becomes the standard design template for future ULABI types.

---

122. Conformance Statement

A conforming implementation MUST NOT claim:

«"ULABI compatible"»

without identifying which ULABI type levels and profiles it implements.

A proper declaration SHOULD look like:

ULABI Implementation Profile

Core Types       ✓
Structural Types ✓
Result Types     ✓
Resource Types   -
Streaming        ✓
Security         ✓
Distributed      -
Tensor           -

This prevents vague compatibility claims.

---

123. Final Architectural Rule

The ULABI Universal Type System exists to establish a common semantic boundary while preserving implementation freedom.

The fundamental model is:

+------------------+
| Programming      |
| Language A       |
+------------------+
         |
         v
+------------------+
| ULABI Type       |
| Contract         |
+------------------+
         |
         v
+------------------+
| Programming      |
| Language B       |
+------------------+

Not:

Language A
    |
    v
Language B

and not:

Language A
    |
    v
Universal Language

but:

Language A
    |
    v
  ULABI
    ^
    |
Language B

---

124. Summary

ULABI's type system provides a universal semantic vocabulary for interoperability without forcing programming languages to become identical.

It defines:

- primitive types;
- structural types;
- algebraic types;
- resource types;
- semantic types;
- type identity;
- canonical representations;
- validation;
- compatibility;
- ownership metadata;
- lifetime metadata;
- mutability;
- resource limits;
- safe conversion;
- streaming;
- zero-copy boundaries;
- distributed type semantics;
- security requirements;
- conformance requirements.

The central principle remains:

«ULABI defines what data means at the interoperability boundary; each language remains free to decide how that data is represented internally.»

Zamani remains Zamani.

Sankofa remains Sankofa.

C remains C.

Rust remains Rust.

Python remains Python.

Every future programming language remains independent.

ULABI is the common contract between them.