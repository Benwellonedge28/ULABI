I inspected the actual Benwellonedge28/ULABI repository before drafting this. The repository already has substantial architecture, ABI, compatibility, distributed, and hardware specifications, while docs/interoperability/cross-language-data.md is currently only a placeholder.

I also checked the existing ULABI-DESIGN.md, which establishes language/runtime/platform neutrality, Minimal Core + Profiles, semantic types, deterministic representations, memory boundaries, distributed interoperability, hardware profiles, compatibility, and conformance.  The repository already has substantial data-types.md, core-abi.md, calling-convention.md, memory-model.md, distributed serialization, distributed ABI, and compatibility specifications, so the document below deliberately assigns responsibilities instead of duplicating them.

1. Exact file to complete first

Primary independent file:

docs/interoperability/cross-language-data.md

This file should be completed before implementing the cross-language data modules, because it establishes the contract those modules implement.

Its responsibility is:

> Define how semantically equivalent data crosses between independently implemented languages without requiring those languages to share their internal type systems, object models, memory managers, compilers, runtimes, or representations.



It should not redefine:

universal primitive types → docs/abi/data-types.md

general ABI → docs/abi/core-abi.md

calling/register/stack conventions → docs/abi/*

memory ownership/lifetime → docs/abi/memory-model.md

wire encoding → docs/distributed/serialization.md

distributed invocation → docs/distributed/remote-calls.md

compatibility negotiation → docs/compatibility/feature-negotiation.md

capability discovery → docs/compatibility/capability-discovery.md


That separation is important because the repository already contains those responsibilities.


---

2. docs/interoperability/cross-language-data.md

Below is the complete specification to copy into that file.

ULABI Cross-Language Data Interoperability Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: "docs/interoperability/cross-language-data.md"
Specification Family: Language Interoperability
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Status: Normative Design Specification / Pre-Implementation
Normative Authorities: "ULABI-DESIGN.md", "ULABI-SPEC.md", "ULABI-VERSIONING.md"
Primary Dependencies: "docs/abi/data-types.md", "docs/abi/core-abi.md", "docs/abi/memory-model.md"
License: Apache-2.0

---

1. Purpose

This specification defines how data is exchanged between independently implemented programming languages through ULABI.

The objective is to allow:

Language A
    |
Compiler A
    |
Runtime A
    |
Language Adapter A
    |
    v
  ULABI
    ^
    |
Language Adapter B
    |
Runtime B
    |
Compiler B
    |
Language B

without requiring Language A and Language B to share:

- source syntax;
- compiler;
- runtime;
- memory manager;
- object model;
- garbage collector;
- ownership system;
- calling convention;
- operating system;
- CPU architecture;
- vendor;
- implementation language.

ULABI defines the interoperability contract.

It does not define the internal representation of participating languages.

---

2. Fundamental Rule

The fundamental rule is:

«Cross-language interoperability is based on semantic contract identity, not implementation representation identity.»

Therefore:

Language A Type
      |
      | adapter
      v
ULABI Type Contract
      |
      | adapter
      v
Language B Type

is valid even when:

A representation != B representation

provided that both representations satisfy the same ULABI contract.

---

3. Scope

This specification defines:

1. cross-language data boundaries;
2. semantic type mapping;
3. type correspondence;
4. representation adaptation;
5. conversion;
6. validation;
7. ownership transfer;
8. borrowing;
9. immutable sharing;
10. mutable sharing;
11. lifetime boundaries;
12. nullability;
13. optional values;
14. errors;
15. records;
16. enums;
17. variants;
18. collections;
19. strings;
20. binary data;
21. handles;
22. capabilities;
23. numeric conversion;
24. precision preservation;
25. lossless conversion;
26. explicitly lossy conversion;
27. object adaptation;
28. opaque values;
29. foreign data;
30. ABI-visible layout;
31. canonical semantic identity;
32. cross-language compatibility;
33. adapter requirements;
34. validation;
35. failure behavior;
36. security requirements;
37. performance requirements;
38. zero-copy boundaries;
39. streaming boundaries;
40. conformance requirements.

This document does NOT define:

- a programming language;
- a universal object model;
- a universal garbage collector;
- a universal memory manager;
- a mandatory wire encoding;
- a network transport;
- distributed service discovery;
- RPC;
- cryptographic algorithms;
- language-specific syntax.

Those responsibilities belong to other ULABI specifications.

---

4. Architectural Position

The ULABI interoperability hierarchy is:

ULABI-DESIGN.md
       |
       v
ULABI-SPEC.md
       |
       v
Core ABI
       |
       +--------------------------+
       |                          |
       v                          v
Universal Data Types       Memory Boundary
       |                          |
       +------------+-------------+
                    |
                    v
          Cross-Language Data
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      FFI      Object Model   Serialization
        |           |           |
        +-----------+-----------+
                    |
                    v
              Implementations

This document consumes the universal data-type contract.

It does not redefine it.

---

5. Language Independence

ULABI MUST remain independent of all programming languages.

The following are equally valid ULABI implementations:

C       -> ULABI
C++     -> ULABI
Rust    -> ULABI
Go      -> ULABI
Java    -> ULABI
Python  -> ULABI
Swift   -> ULABI
Kotlin  -> ULABI
Fortran -> ULABI
Ada     -> ULABI
Zamani  -> ULABI
Sankofa -> ULABI

No language is the reference language.

No language may define the semantics of ULABI.

Zamani and Sankofa remain independent projects.

ULABI MUST NOT assume that either is present.

---

6. Semantic Identity

A cross-language value MUST be identified by its ULABI semantic contract.

For example:

ULABI Type:
com.example.user/UserID/1

may be represented internally as:

Language A:
struct UserID {
    uint64 value;
}

and:

Language B:
class UserID {
    BigInteger value;
}

The internal representations may differ.

They are interoperable only if both satisfy the ULABI "UserID" contract.

---

7. Type Correspondence

A language adapter MUST maintain an explicit mapping:

Source Language Type
        |
        v
ULABI Type
        |
        v
Target Language Type

Conceptually:

TypeMapping {
    source_type
    ulabi_type
    target_type
    conversion_policy
    ownership_policy
    lifetime_policy
    nullability_policy
    loss_policy
}

The mapping MUST NOT be inferred solely from names.

For example:

String

in two languages does not automatically establish semantic equivalence.

---

8. Type Mapping Rules

A mapping MAY be:

8.1 Identity Mapping

The language representation already satisfies the ULABI representation.

Language Type
      =
ULABI Type

8.2 Adapted Mapping

The language representation requires conversion.

Language Type
      |
 Conversion
      v
ULABI Type

8.3 Opaque Mapping

The value cannot safely be represented directly.

Language Type
      |
      v
ULABI Opaque Handle

8.4 Unsupported Mapping

No valid mapping exists.

The adapter MUST reject the operation.

---

9. No Unsafe Implicit Equivalence

An adapter MUST NOT assume equivalence based only on:

- type names;
- source-language keywords;
- memory size;
- field count;
- field order;
- pointer width;
- ABI coincidence;
- compiler behavior.

For example:

LanguageA::String

and:

LanguageB::String

MUST NOT automatically be considered equivalent.

---

10. Primitive Values

Primitive values SHOULD map directly where their semantics and representation are compatible.

Examples:

ULABI Bool
ULABI I32
ULABI I64
ULABI U32
ULABI U64
ULABI F32
ULABI F64
ULABI Byte
ULABI Char

An adapter MUST verify:

- width;
- signedness;
- range;
- representation;
- alignment where applicable;
- conversion behavior.

---

11. Integer Conversion

Integer conversion MUST explicitly identify:

source width
source signedness
target width
target signedness
range
overflow policy

A conversion is lossless when every source value has an exact target representation.

Example:

I32 -> I64

is normally lossless.

Example:

I64 -> I32

is potentially lossy.

An implementation MUST NOT silently truncate a value unless the contract explicitly permits truncation.

---

12. Integer Overflow

If a target type cannot represent a source value, the adapter MUST follow the declared conversion policy.

Permitted policies include:

Reject
ReturnError
Saturate
ExplicitWrap
ExplicitTruncate

The default cross-language policy SHOULD be:

Reject

Silent truncation MUST NOT be the default.

---

13. Floating-Point Conversion

Floating-point conversion MUST define:

- precision;
- range;
- NaN behavior;
- infinity behavior;
- signed-zero behavior;
- rounding;
- overflow;
- underflow.

For example:

F64 -> F32

MAY lose precision.

The adapter MUST NOT claim that this is lossless.

---

14. Lossless Conversion

A conversion SHOULD be classified as one of:

Lossless
ConditionallyLossless
Lossy
Unsupported

Conceptually:

ConversionResult {
    status
    value
    diagnostics
}

A consumer MUST be able to determine whether semantic information was lost.

---

15. Lossy Conversion

Lossy conversion MAY be permitted only when:

1. the contract permits it;
2. the caller explicitly requests it;
3. the conversion policy identifies the loss;
4. validation succeeds.

Lossy conversion MUST NOT silently occur merely because the target language has fewer capabilities.

---

16. Strings

ULABI strings are semantic Unicode strings.

A language adapter MUST explicitly map:

Language String
       |
       v
ULABI String

The adapter MUST account for:

- encoding;
- validity;
- length;
- normalization;
- embedded terminators;
- maximum size.

Null termination MUST NOT be assumed.

---

17. String Encoding

The canonical ULABI text encoding is defined by the applicable ULABI data-type and serialization specifications.

A language MAY internally use:

- UTF-8;
- UTF-16;
- UTF-32;
- compact strings;
- ropes;
- immutable string objects;
- reference-counted strings;
- garbage-collected strings.

These internal choices MUST NOT alter ULABI semantics.

---

18. Bytes

Binary data MUST remain distinct from text.

String != Bytes

An adapter MUST NOT automatically convert arbitrary bytes to text.

Explicit conversion MAY be provided when the contract identifies:

- encoding;
- validation;
- error policy.

---

19. Records

A ULABI Record consists of semantically identified fields.

Example:

User {
    id: UserID
    name: String
    active: Bool
}

A language MAY represent it as:

struct
class
record
object
tuple
map
dictionary
generated type

provided the adapter preserves the ULABI field contract.

---

20. Field Identity

Field identity MUST be stable.

Field identity MUST NOT depend solely upon:

- declaration order;
- source-language field name;
- memory offset;
- compiler-generated symbol.

The ULABI field identity is authoritative.

---

21. Field Evolution

When a record evolves:

Version 1:
id
name

Version 2:
id
name
email

the new field MAY be compatible if the schema defines it as optional or otherwise provides a valid compatibility mechanism.

Changing:

name: String

to:

name: Integer

is not automatically compatible.

---

22. Unknown Fields

Unknown fields MUST follow the applicable schema policy.

Possible policies:

Reject
Ignore
Preserve
Forward
RequireNegotiation

An adapter acting as a transparent intermediary MUST preserve unknown fields when the contract requires preservation.

---

23. Enums

ULABI enum variants MUST have stable identities.

A language may represent an enum as:

integer
symbol
tagged union
class hierarchy
object

The adapter MUST preserve the ULABI variant identity.

---

24. Unknown Enum Variants

Unknown variants MUST NOT be silently mapped to an unrelated known variant.

Possible policies:

Preserve
Unknown
Reject
Negotiate

The selected policy MUST be explicit.

---

25. Variants

A ULABI Variant identifies one of several semantic alternatives.

Example:

Value =
    Integer(I64)
    Text(String)
    Binary(Bytes)

The active variant MUST be explicit.

A language adapter MUST NOT infer the active variant from arbitrary memory layout.

---

26. Option

ULABI "Option<T>" has:

None
Some(value)

The adapter MUST preserve the distinction between:

None

and:

Some(0)
Some("")
Some(empty_bytes)

A language's "null", "nil", "None", "optional", or equivalent MAY be mapped to ULABI "Option", but the mapping MUST be explicit.

---

27. Result

ULABI "Result<T,E>" has:

Success(value)
Failure(error)

Languages may implement this using:

- exceptions;
- error values;
- status codes;
- tagged unions;
- result objects;
- algebraic data types.

The adapter MUST preserve the semantic distinction.

---

28. Exceptions

Exceptions are language-specific.

ULABI MUST NOT require all languages to expose exceptions.

When an exception crosses a ULABI boundary, it MUST be adapted to the ULABI error/result contract.

A language-specific exception object MUST NOT automatically cross the boundary as an opaque object unless explicitly permitted.

---

29. Errors

Errors crossing language boundaries MUST have stable semantic identity.

Conceptually:

Error {
    error_id
    category
    message
    details
    retryability
    severity
}

The precise error contract is governed by the applicable ULABI error specifications.

This document only defines the cross-language mapping responsibility.

---

30. Collections

Collections MAY be mapped between languages.

Examples:

ULABI List<T>

may map to:

array
vector
list
slice
sequence
collection
iterator-backed structure

The adapter MUST preserve:

- ordering;
- element semantics;
- cardinality;
- mutation semantics;
- ownership;
- lifetime.

---

31. Collection Mutation

If a source collection is immutable:

Immutable List<T>

the adapter MUST NOT expose it as mutable unless a copy or explicit ownership transition occurs.

Likewise, a mutable source MUST NOT be exposed as independently mutable through a boundary that promises immutability.

---

32. Ownership

Ownership semantics MUST be explicit.

Possible states include:

Owned
Borrowed
Shared
ImmutableShared
Transferred
Moved
Released

A language's ownership model MAY differ completely from another language's model.

ULABI defines only the boundary contract.

---

33. Ownership Transfer

When ownership is transferred:

Language A
   |
   | transfer
   v
ULABI Boundary
   |
   v
Language B

the source MUST no longer use the value according to the previous ownership contract.

The adapter MUST enforce the transition where the source language permits enforcement.

---

34. Borrowing

A borrowed value MUST have an explicit lifetime.

The target MUST NOT retain a borrowed reference after the declared lifetime expires.

If safe borrowing cannot be established, the adapter MUST:

copy
retain
transfer ownership
or reject

It MUST NOT create an invalid dangling reference.

---

35. Shared Values

Shared values require explicit synchronization semantics when mutable.

The contract MUST define whether sharing is:

Immutable
ReadOnly
MutableExclusive
MutableSynchronized
Atomic
Unspecified

Unspecified mutable sharing MUST NOT be assumed safe.

---

36. Zero-Copy

ULABI MAY permit zero-copy exchange.

Zero-copy MUST NOT mean:

«expose arbitrary foreign memory directly.»

Zero-copy is valid only when:

1. layout is compatible;
2. alignment is compatible;
3. ownership is safe;
4. lifetime is safe;
5. mutability is compatible;
6. security requirements are satisfied;
7. architecture requirements are satisfied.

Otherwise, the adapter MUST copy or reject.

---

37. Copy Fallback

Every zero-copy optimization SHOULD have a safe copy fallback when the contract permits it.

Conceptually:

Compatible?
   |
 +--+--+
 |     |
YES    NO
 |     |
Zero   Copy
Copy

If neither zero-copy nor copying can satisfy the contract, the operation MUST fail explicitly.

---

38. Handles

Resources that cannot safely cross as ordinary values MUST be represented as ULABI handles.

Examples:

File
Device
Socket
GPU resource
Memory region
Database connection
Process
Service

Raw pointers MUST NOT be treated as portable handles.

---

39. Capabilities

Capabilities MUST remain distinct from ordinary data.

A language adapter MUST NOT manufacture authority merely by converting a value.

For example:

Bytes

MUST NOT automatically become:

FileCapability

A capability MUST be explicitly granted by the applicable security contract.

---

40. Opaque Types

An implementation MAY expose an opaque type:

Opaque<T>

when its internal representation cannot safely be standardized.

The opaque value MUST have:

- stable type identity;
- explicit ownership;
- lifetime;
- permitted operations;
- capability requirements;
- destruction/release semantics.

---

41. Object Interoperability

ULABI MUST NOT require languages to share one universal object model.

A source object may be adapted into:

Record
Handle
Interface
Opaque
Variant

depending on the semantic contract.

Language-specific inheritance, vtables, method dispatch, object headers, and runtime metadata MUST NOT be exposed as universal semantics unless a dedicated profile defines them.

---

42. Interface Objects

A cross-language interface SHOULD be represented through explicit ULABI interface identity.

Conceptually:

Interface {
    interface_id
    version
    methods[]
    capabilities
    effects
}

Method signatures MUST use ULABI types.

---

43. Generic Types

Generic types MAY cross a ULABI boundary when their instantiated form has an explicit ULABI contract.

For example:

List<User>

MUST identify:

List
User
versions
constraints

A language's compile-time generic machinery MUST NOT be assumed to exist in another language.

---

44. Type Constraints

Generic constraints MAY be represented as ULABI metadata.

Examples:

Comparable
Serializable
Hashable
Numeric
Ordered

Constraints MUST have semantic definitions.

Language-specific traits/interfaces MUST NOT automatically become ULABI constraints.

---

45. Dynamic Values

ULABI MAY support dynamic values through a tagged semantic representation.

Conceptually:

DynamicValue {
    type_id
    version
    payload
}

The receiver MUST validate the declared type before interpreting the payload.

---

46. Type Validation

Every cross-language adapter MUST validate incoming values before exposing them to the target language.

Validation MUST cover, where applicable:

- type identity;
- version;
- representation;
- range;
- nullability;
- required fields;
- ownership;
- lifetime;
- size;
- alignment;
- capability requirements;
- security constraints.

---

47. Validation Failure

Validation failure MUST NOT result in undefined behavior.

The adapter MUST produce an explicit failure.

Possible outcomes:

TypeMismatch
InvalidValue
UnsupportedVersion
RangeViolation
OwnershipViolation
LifetimeViolation
SecurityViolation
UnsupportedConversion
ResourceLimit

The exact error taxonomy is defined by the applicable ULABI error specification.

---

48. Adapter Isolation

A language adapter MUST isolate language-specific semantics from ULABI semantics.

Architecture:

Language Runtime
      |
      v
Language Adapter
      |
      v
ULABI Contract
      |
      v
Language Adapter
      |
      v
Target Runtime

Language-specific implementation details MUST terminate at the adapter boundary.

---

49. Adapter Responsibilities

An adapter is responsible for:

1. type mapping;
2. conversion;
3. validation;
4. ownership handling;
5. lifetime handling;
6. error mapping;
7. capability enforcement;
8. representation adaptation;
9. compatibility verification;
10. resource cleanup;
11. diagnostics;
12. conformance reporting.

---

50. Adapter Non-Responsibilities

An adapter MUST NOT redefine:

- ULABI type semantics;
- ULABI version semantics;
- ULABI security policy;
- transport semantics;
- distributed consensus;
- service discovery;
- application-specific business semantics.

---

51. Memory Representation

A language's native memory layout MUST NOT automatically become a ULABI representation.

For example:

struct {
    field_a;
    field_b;
}

does not automatically define:

ULABI Record

The ULABI contract determines the boundary representation.

---

52. ABI Layout

When a ULABI profile explicitly requires ABI-visible layout, the layout MUST be completely specified.

It MUST identify, where applicable:

- field offsets;
- field sizes;
- alignment;
- padding;
- endianness;
- representation;
- pointer semantics;
- ownership;
- lifetime.

Undefined native padding MUST NOT cross the boundary.

---

53. Architecture Independence

Cross-language data MUST remain portable across architectures unless an architecture-specific profile is explicitly selected.

The contract MUST NOT silently depend on:

sizeof(int)
sizeof(long)
pointer width
native endianness
native alignment
native ABI

---

54. Distributed Boundaries

When cross-language data crosses a distributed boundary, this specification defines the semantic mapping.

The serialization specification defines the actual serialization contract.

The distributed ABI defines the invocation semantics.

Therefore:

Cross-Language Data
        |
        v
Serialization
        |
        v
Transport
        |
        v
Distributed ABI

This document MUST NOT duplicate the distributed serialization protocol.

---

55. Local Boundaries

For in-process interoperability:

Language A
    |
    v
ULABI Adapter
    |
    v
Language B

the implementation MAY optimize using:

- direct calls;
- shared memory;
- views;
- zero-copy;
- native-compatible representations.

Such optimizations MUST preserve the same semantic contract as the portable path.

---

56. Process Boundaries

Across processes:

Process A
    |
ULABI Adapter
    |
Boundary
    |
ULABI Adapter
    |
Process B

the implementation MUST NOT rely on:

- process-local pointers;
- process-local addresses;
- language runtime object identities;
- thread-local state.

---

57. Streaming Data

Large values MAY be exchanged as streams.

A streaming adapter MUST define:

- element type;
- ordering;
- completion;
- cancellation;
- failure;
- ownership;
- backpressure;
- resource limits.

Streaming semantics belong to the applicable ULABI streaming/runtime profiles.

This document defines only the cross-language data mapping.

---

58. Incremental Conversion

An adapter MAY convert data incrementally.

Example:

Language A Stream
        |
        v
ULABI Stream<T>
        |
        v
Language B Stream<T>

The adapter MUST NOT require full materialization when the contract explicitly permits streaming.

---

59. Resource Limits

Adapters MUST enforce configured limits.

Possible limits include:

maximum value size
maximum string length
maximum collection length
maximum nesting depth
maximum conversion memory
maximum conversion time
maximum object count

A hostile or malformed value MUST NOT cause uncontrolled resource consumption.

---

60. Security

Cross-language data is untrusted at the boundary unless explicitly established otherwise.

Adapters MUST validate before interpretation.

Adapters MUST NOT:

- execute code embedded in ordinary data;
- trust foreign object metadata;
- trust foreign pointers;
- trust foreign allocator state;
- trust foreign runtime references;
- create capabilities implicitly.

---

61. Deserialization Safety

When data enters a language runtime, the adapter MUST prevent unsafe construction of runtime objects.

Language-specific object deserialization mechanisms MUST NOT automatically be considered safe ULABI mappings.

---

62. Code Execution Prohibition

ULABI data MUST be data.

Executable behavior MUST require an explicitly defined interface or capability.

A serialized value MUST NOT implicitly execute:

- constructors;
- methods;
- scripts;
- macros;
- arbitrary callbacks;
- language runtime hooks.

---

63. Determinism

Equivalent semantic values MUST have equivalent observable ULABI meaning.

Conversion MUST NOT introduce hidden nondeterministic semantics.

If a conversion inherently depends on:

- locale;
- timezone;
- environment;
- runtime state;
- random state;

the contract MUST identify that dependency.

---

64. Time and Locale

Language-specific date/time representations MUST NOT be assumed equivalent.

For example:

Date
Timestamp
Duration
LocalDate
ZonedDateTime

MUST have explicit semantic mappings.

Locale-dependent representations MUST declare their locale semantics.

---

65. Numeric Semantics

Numeric conversions MUST preserve semantic meaning whenever possible.

The adapter MUST distinguish:

exact
rounded
saturated
truncated
overflowed
unsupported

A target-language numeric type MUST NOT silently change the mathematical meaning of a value.

---

66. Decimal and Arbitrary Precision

If a language supports arbitrary precision and the ULABI contract supports it, conversion MAY preserve arbitrary precision.

If the target language cannot represent the value exactly, the adapter MUST either:

preserve using an extension representation
convert explicitly
or reject

Silent precision loss is prohibited by default.

---

67. Semantic Units

Values carrying physical or domain units SHOULD preserve unit identity.

For example:

Length<meter>

MUST NOT silently become:

Length<foot>

without an explicit conversion.

Unit semantics belong to the applicable ULABI units specification.

---

68. Security-Sensitive Types

Security-sensitive values MUST receive special handling.

Examples:

Credential
Secret
Key
Capability
Token
Identity
AuthenticationContext

Adapters MUST NOT accidentally convert these into ordinary strings or byte arrays when doing so would weaken the security contract.

---

69. Identity

An object identity is not automatically transferable across languages.

The adapter MUST distinguish:

Value Equality

from:

Object Identity

and:

Resource Identity

These MUST NOT be conflated.

---

70. Equality

ULABI equality semantics MUST be determined by the type contract.

Possible equality models include:

ValueEqual
IdentityEqual
SemanticEqual
CanonicalEqual

A language's native equality operator MUST NOT automatically define ULABI equality.

---

71. Hashing

Language-specific hash functions MUST NOT define ULABI semantic identity.

If a type is hashable across a ULABI boundary, the applicable contract MUST define the semantic hashing requirements.

---

72. Mutable Objects

Mutable objects crossing a language boundary require explicit synchronization semantics.

An adapter MUST NOT expose:

Language A mutable object

as:

Language B mutable object

without an explicit shared-mutation contract.

Safe alternatives include:

copy
immutable view
exclusive transfer
synchronized handle

---

73. Callback Data

Callback parameters MUST use ULABI types.

A callback MUST NOT assume that the target runtime understands the source language's native types.

Callback lifetime and cancellation MUST follow the applicable runtime/async specifications.

---

74. Function Values

A function value MAY be represented as a ULABI interface or callback handle.

A raw language-specific function pointer MUST NOT automatically become a portable ULABI function value.

The function contract MUST define:

- interface identity;
- parameters;
- return value;
- errors;
- effects;
- capabilities;
- lifetime;
- invocation mode.

---

75. Closures

Language closures commonly capture runtime-specific state.

A closure MUST NOT cross a ULABI boundary as a raw object.

It MUST be represented using an explicit ULABI callable contract if supported.

Captured resources MUST have explicit ownership and capability semantics.

---

76. Generational Compatibility

An adapter MUST support compatibility between independently versioned language bindings.

The compatibility process is:

Source Type
    |
Version
    |
ULABI Type
    |
Compatibility Check
    |
Target Type

The adapter MUST NOT assume that matching source-language versions imply ULABI compatibility.

---

77. Schema Evolution

Cross-language schemas SHOULD evolve independently from language releases.

A language release MUST NOT automatically create a new ULABI type identity unless the semantic contract changed.

---

78. Compatibility Matrix

An adapter SHOULD be able to report:

Source Type
Target Type
ULABI Type
Compatible
Conditionally Compatible
Lossy
Unsupported

Example:

Source: RustString
ULABI: String/1
Target: JavaString
Status: Compatible

---

79. Diagnostics

Conversion failures SHOULD provide structured diagnostics.

Conceptually:

ConversionDiagnostic {
    source_type
    target_type
    ulabi_type
    reason
    location
    severity
    recoverable
}

Diagnostics MUST NOT expose secrets or sensitive memory.

---

80. Error Recovery

When conversion fails, the adapter MAY:

Retry with negotiated representation
Use safe fallback
Copy instead of zero-copy
Use opaque handle
Return explicit error

It MUST NOT silently reinterpret incompatible data.

---

81. No Semantic Guessing

The adapter MUST NOT "guess" the intended meaning of ambiguous data.

For example:

123

MUST NOT automatically be interpreted as:

UserID
Timestamp
Money
FileDescriptor
Temperature

without the contract establishing the semantic type.

---

82. Application Semantics

Application-specific meaning belongs to the application's ULABI schema.

For example:

UInt64

does not itself mean:

UserID

The application contract establishes that meaning.

---

83. Canonical Internal Boundary

Implementations SHOULD use a common internal ULABI boundary representation where doing so reduces adapter complexity.

Conceptually:

Language A
    |
Adapter A
    |
ULABI Internal Representation
    |
Adapter B
    |
Language B

This is an implementation optimization, not a requirement.

---

84. Adapter Composition

Adapters MAY be composed.

For example:

Language A
    |
Adapter A
    |
ULABI
    |
Adapter B
    |
Language B

The architecture MUST avoid:

A -> B
A -> C
A -> D
A -> E
...

as the only interoperability model.

ULABI exists specifically to provide a common contract.

---

85. N-to-N Interoperability

With ULABI:

N languages

should ideally require:

N ULABI adapters

rather than:

N × N direct language integrations

This is one of the principal architectural benefits of ULABI.

---

86. Adapter Versioning

Each adapter MUST identify:

language identity
language version
adapter version
ULABI version
supported profiles
supported types

Conceptually:

AdapterMetadata {
    language_id
    language_version
    adapter_version
    ulabi_version
    profiles[]
}

---

87. Capability Discovery

Adapters MAY expose supported data capabilities.

For example:

Types:
String
Bytes
Record
Variant

Profiles:
Streaming
ZeroCopy
SharedMemory
Tensor
Security

Capability discovery MUST use the dedicated ULABI capability-discovery mechanism.

This document defines what cross-language data capabilities MAY be reported; it does not redefine discovery protocol semantics.

---

88. Feature Negotiation

When two implementations support different representations, they MAY negotiate a common representation.

Example:

A supports:
String/1
String/2

B supports:
String/1

Negotiated:
String/1

Feature negotiation MUST use the dedicated ULABI negotiation specification.

---

89. Graceful Degradation

When an optimization is unavailable:

Zero-copy
   |
unsupported
   v
Copy

is acceptable if the semantic contract remains intact.

However:

Exact Integer
   |
unsupported
   v
Truncated Integer

is NOT acceptable unless explicitly authorized.

---

90. Security Boundary

The adapter is a security boundary.

The security model MUST assume:

Foreign Data = Untrusted

until validation establishes otherwise.

The adapter MUST enforce the applicable:

- type restrictions;
- resource limits;
- capability restrictions;
- memory safety;
- validation rules.

---

91. Allocator Independence

A language MUST NOT free foreign memory using an allocator it does not own unless the contract explicitly defines a compatible allocator boundary.

Examples of safe models:

Producer-owned memory
Consumer-owned memory
Shared allocator
Explicit release callback
Reference-counted resource
Copy

The model MUST be explicit.

---

92. Lifetime Independence

A target language MUST NOT retain a reference after the source lifetime expires.

If retention is required, the adapter MUST perform an explicit lifetime transition:

Borrow
   |
   v
Retain
   |
   v
Owned/Shared

or:

Copy

---

93. Thread Safety

Thread-safety semantics MUST be explicit.

A language adapter MUST NOT assume that a foreign object is thread-safe merely because it is represented as a ULABI value.

The applicable contract MUST identify:

ThreadSafe
ThreadCompatible
SingleThreadOnly
ActorBound
ExternallySynchronized

where applicable.

---

94. Reentrancy

If a ULABI call can re-enter the source runtime, the adapter MUST document the reentrancy semantics.

A language runtime that is not reentrant MUST be protected from unsupported reentrant calls.

---

95. Concurrency

Cross-language data exchange MUST preserve the semantics defined by the concurrency/runtime profiles.

Data conversion itself MUST NOT silently introduce races.

---

96. Testing Requirements

Every conformant adapter MUST test at minimum:

Primitive Types

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

Structured Types

Record
Enum
Variant
Option
Result

Data

String
Bytes
List
Map

Boundary Conditions

minimum values
maximum values
zero
empty
null/None
large values
invalid values
unknown fields
unknown variants
overflow
underflow

---

97. Round-Trip Testing

Where a representation is reversible:

Language A
   |
   v
ULABI
   |
   v
Language B
   |
   v
ULABI
   |
   v
Language A

the semantic value MUST remain equivalent.

Tests MUST compare semantic identity rather than native object identity.

---

98. Differential Testing

At least two independent implementations SHOULD be tested against the same ULABI corpus.

For example:

Implementation A
        |
        v
Test Corpus
        ^
        |
Implementation B

Differences MUST be investigated.

---

99. Property Testing

Adapters SHOULD use property-based testing for:

- integer conversion;
- collection conversion;
- Unicode;
- optional values;
- variants;
- schema evolution;
- ownership transitions;
- boundary sizes.

---

100. Fuzz Testing

Cross-language adapters SHOULD be fuzz tested with:

- malformed values;
- invalid encodings;
- oversized values;
- deeply nested values;
- invalid type IDs;
- invalid versions;
- duplicate fields;
- invalid discriminants;
- malicious handles.

A crash, memory corruption, capability escalation, or undefined behavior is a conformance failure.

---

101. Conformance

An implementation claiming ULABI cross-language data conformance MUST satisfy:

Type Mapping
       ✓
Validation
       ✓
Conversion
       ✓
Ownership
       ✓
Lifetime
       ✓
Error Mapping
       ✓
Security
       ✓
Compatibility
       ✓
Resource Limits
       ✓
Conformance Tests
       ✓

---

102. Conformance Levels

The cross-language data profile SHOULD eventually define:

Level 0 — Core Scalars

Primitive values.

Level 1 — Structured Data

Records, enums, variants.

Level 2 — Collections

Lists, maps, sets, tuples.

Level 3 — Ownership and Memory

Borrowing, transfer, shared/immutable data.

Level 4 — Advanced Interoperability

Handles, opaque types, callbacks, streaming, zero-copy.

Level 5 — Distributed Interoperability

Cross-process and distributed semantic exchange.

An implementation MUST identify the level it supports.

---

103. Reference Adapter Model

The reference implementation SHOULD expose:

TypeRegistry
TypeMapper
Converter
Validator
OwnershipManager
LifetimeManager
ErrorMapper
HandleManager
CapabilityGuard
CompatibilityChecker
DiagnosticReporter

These are implementation modules, not programming-language requirements.

---

104. Required Invariants

The following invariants are mandatory:

Invariant 1

A ULABI type MUST have stable semantic identity.

Invariant 2

A language-specific type MUST NOT automatically become a ULABI type.

Invariant 3

Unsafe conversion MUST fail explicitly.

Invariant 4

Lossy conversion MUST be identifiable.

Invariant 5

Ownership MUST NOT be ambiguous.

Invariant 6

A borrowed value MUST NOT outlive its declared lifetime.

Invariant 7

Raw foreign pointers MUST NOT become portable references.

Invariant 8

Capabilities MUST NOT be created by data conversion.

Invariant 9

Architecture-specific representation MUST NOT silently become portable representation.

Invariant 10

Unknown semantic values MUST NOT silently become unrelated known values.

Invariant 11

Security validation MUST occur before unsafe interpretation.

Invariant 12

Optimizations MUST preserve the portable semantic contract.

---

105. Failure Model

Possible failures include:

TypeMismatch
UnsupportedType
UnsupportedVersion
InvalidValue
RangeViolation
PrecisionLoss
EncodingMismatch
SchemaMismatch
OwnershipViolation
LifetimeViolation
MutationViolation
CapabilityViolation
SecurityViolation
ResourceLimit
UnsupportedConversion
InvalidHandle
ConcurrencyViolation

Failures MUST be explicit and diagnosable.

---

106. Recovery

Safe recovery MAY include:

Copy
Retry
Alternative representation
Older compatible schema
Negotiated feature
Opaque handle
Fallback conversion

Recovery MUST NOT silently weaken:

- security;
- type safety;
- ownership;
- lifetime;
- semantic correctness.

---

107. Performance

Implementations SHOULD optimize:

Primitive pass-through
Zero-copy
Batch conversion
Streaming
Arena allocation
Shared immutable data
Cached type mappings
Compiled conversion plans

Performance optimizations MUST remain semantically equivalent to the portable implementation.

---

108. No Language-Specific Leakage

A conformant ULABI implementation MUST NOT expose language-specific assumptions as universal requirements.

For example, ULABI MUST NOT require:

Rust ownership
C ABI
C++ vtables
Java GC
Python objects
Zamani semantics
Sankofa semantics

as universal mechanisms.

---

109. Reference Architecture

The recommended architecture is:

                    +----------------------+
                    |   ULABI Type Schema  |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |    Type Registry     |
                    +----------+-----------+
                               |
               +---------------+---------------+
               |                               |
               v                               v
      +----------------+             +----------------+
      | Language A     |             | Language B     |
      | Adapter        |             | Adapter        |
      +-------+--------+             +-------+--------+
              |                              |
              v                              v
       +-------------+                +-------------+
       | Converter   |                | Converter   |
       +------+------+                +------+------+
              |                              |
              v                              v
       +-------------+                +-------------+
       | Validator   |                | Validator   |
       +------+------+                +------+------+
              |                              |
              +--------------+---------------+
                             |
                             v
                    ULABI Semantic Contract

---

110. Integration With Other Specifications

This specification integrates with:

docs/abi/data-types.md

for universal type definitions.

docs/abi/core-abi.md

for ABI boundary semantics.

docs/abi/memory-model.md

for memory, ownership, and lifetime.

docs/interoperability/language-interoperability.md

for language-level interoperability.

docs/interoperability/foreign-function-interface.md

for native function integration.

docs/interoperability/object-model.md

for object/interface adaptation.

docs/interoperability/name-mangling.md

for symbol identity.

docs/interoperability/symbol-resolution.md

for symbol binding.

docs/distributed/serialization.md

for serialized representation.

docs/distributed/distributed-abi.md

for distributed execution boundaries.

docs/distributed/remote-calls.md

for remote invocation.

docs/distributed/distributed-errors.md

for distributed failure semantics.

docs/compatibility/backwards-compatibility.md

for backward compatibility.

docs/compatibility/forwards-compatibility.md

for forward compatibility.

docs/compatibility/feature-negotiation.md

for representation negotiation.

docs/compatibility/capability-discovery.md

for capability discovery.

docs/compatibility/graceful-degradation.md

for safe fallback.

docs/security/security-model.md

for security requirements.

docs/runtime/resource-management.md

for resource lifecycle.

docs/runtime/async-model.md

for asynchronous values and callbacks.

docs/standards/conformance.md

for certification of implementations.

docs/standards/test-suite.md

for executable conformance testing.

---

111. Responsibility Boundary

The responsibility split MUST remain:

                     ULABI
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
  Type Semantics   ABI Boundary   Language Mapping
       |               |               |
data-types.md     core-abi.md    cross-language-data.md
                                       |
                 +---------------------+--------------------+
                 |                     |                    |
                 v                     v                    v
             FFI/Object          Memory/Ownership      Serialization

Cross-language-data MUST coordinate these systems.

It MUST NOT replace them.

---

112. Implementation Independence

A ULABI adapter MAY be implemented in any language.

Possible implementations include:

C
C++
Rust
Go
Java
Python
Swift
Kotlin
Zamani
Sankofa

The implementation language MUST NOT alter the normative ULABI contract.

---

113. Final Architectural Rule

The central rule of this specification is:

«ULABI defines the semantic contract at the boundary; each language remains free to implement that contract internally according to its own language, runtime, memory, and execution model.»

Therefore:

Language A
    !=
Language B

Language A
    !=
ULABI

Language B
    !=
ULABI

but:

Language A
    |
    v
ULABI Contract
    ^
    |
Language B

This is the foundation of language-independent interoperability.

---

114. Completion Criteria

This document is complete when:

- semantic mapping is specified;
- type correspondence is specified;
- conversion rules are specified;
- lossless/lossy behavior is specified;
- ownership is specified;
- lifetime is specified;
- validation is specified;
- security is specified;
- zero-copy constraints are specified;
- opaque values are specified;
- handles are specified;
- compatibility is specified;
- distributed boundaries are explicitly delegated;
- serialization is explicitly delegated;
- conformance requirements are specified;
- implementation module boundaries are defined.

No later document should need to redefine these cross-language data responsibilities.
---

3. What belongs in which file

The important thing for your workflow is not to edit this file later just because another specification is implemented.

I would lock the responsibility matrix now.

File	Owns	Must NOT own

docs/interoperability/cross-language-data.md	Semantic mapping between language types and ULABI types	Wire encoding, RPC, language syntax
docs/abi/data-types.md	Universal ULABI type definitions	Language-specific mappings
docs/abi/core-abi.md	Core ABI boundary	Language adapters
docs/abi/memory-model.md	Ownership, lifetime, memory boundaries	Language-specific type conversion
docs/interoperability/language-interoperability.md	Overall language interoperability model	Detailed value conversion
docs/interoperability/foreign-function-interface.md	FFI/native function boundary	Universal serialization
docs/interoperability/object-model.md	Object/interface adaptation	Primitive type definitions
docs/interoperability/name-mangling.md	Symbol naming	Data representation
docs/interoperability/symbol-resolution.md	Symbol lookup/binding	Type conversion
docs/distributed/serialization.md	Serialized representation	Language mapping
docs/distributed/distributed-abi.md	Distributed execution	Data-type semantics
docs/distributed/remote-calls.md	Remote invocation	Type definitions
docs/distributed/distributed-errors.md	Distributed failures	General type mapping
docs/compatibility/backwards-compatibility.md	Backward compatibility	Conversion implementation
docs/compatibility/forwards-compatibility.md	Forward compatibility	Serialization
docs/compatibility/feature-negotiation.md	Negotiating capabilities/features	Type implementation
docs/compatibility/capability-discovery.md	Discovering supported capabilities	Conversion algorithms
docs/security/security-model.md	Overall security contract	Language-specific conversion
docs/runtime/resource-management.md	Runtime resources/lifetimes	Type identity
docs/standards/conformance.md	What counts as conformant	Implementation details
docs/standards/test-suite.md	Executable tests	Normative architecture


This separation is particularly important because docs/distributed/serialization.md already establishes that serialization is a separate concern from distributed invocation, service discovery, and compatibility.


---

4. Implementation files/modules

The current repository is still primarily specification-oriented; the repository tree shows the large specification families, but this cross-language implementation layer is not yet a completed src/ subsystem.

When implementation begins, do not create one giant cross_language.rs/.cpp/.go file.

Use these module boundaries.

Independent foundation modules — implement first

These should be completed without depending on the higher-level adapter modules:

src/
└── interop/
    ├── type_identity.*
    ├── type_descriptor.*
    ├── type_registry.*
    ├── semantic_type.*
    ├── conversion_policy.*
    ├── conversion_result.*
    ├── diagnostics.*
    └── validation.*

1. type_identity.*

Owns:

namespace;

type ID;

version;

stable identity comparison.


2. semantic_type.*

Owns the language-independent semantic type abstraction.

3. type_descriptor.*

Owns machine-readable type metadata.

4. type_registry.*

Owns registration and lookup of ULABI types.

5. conversion_policy.*

Owns:

lossless;

lossy;

reject;

saturation;

truncation;

fallback policies.


6. conversion_result.*

Owns structured conversion outcomes.

7. validation.*

Owns boundary validation.

8. diagnostics.*

Owns structured interoperability diagnostics.

These eight are the first implementation layer.


---

5. Second implementation layer

After the foundation:

src/interop/
├── type_mapping.*
├── converter.*
├── primitive_converter.*
├── numeric_converter.*
├── string_converter.*
├── bytes_converter.*
├── collection_converter.*
├── record_converter.*
├── enum_converter.*
├── variant_converter.*
├── option_converter.*
└── result_converter.*

These modules implement actual semantic transformations.


---

6. Memory/ownership layer

Then:

src/interop/memory/
├── ownership.*
├── lifetime.*
├── borrow.*
├── transfer.*
├── shared_value.*
├── immutable_view.*
├── allocator_boundary.*
└── zero_copy.*

These modules integrate with docs/abi/memory-model.md.

They should not redefine the memory model.

They implement it.


---

7. Resource/opaque layer

Then:

src/interop/resource/
├── handle.*
├── opaque_value.*
├── resource_lifetime.*
├── release.*
└── capability_guard.*

These implement:

ULABI Handle
ULABI Opaque
ULABI Capability

without turning arbitrary foreign objects into trusted resources.


---

8. Adapter layer

Only after the common machinery is complete:

src/interop/adapter/
├── adapter.*
├── adapter_metadata.*
├── adapter_registry.*
├── language_binding.*
├── mapping_plan.*
├── conversion_plan.*
└── compatibility_checker.*

The architecture becomes:

Language Binding
      |
Adapter
      |
Mapping Plan
      |
Converter
      |
Validator
      |
ULABI Type


---

9. Language-specific modules

Language integrations should be isolated.

For example:

implementations/
├── c/
│   └── ulabi_adapter.*
├── cpp/
│   └── ulabi_adapter.*
├── rust/
│   └── ulabi_adapter.*
├── go/
│   └── ulabi_adapter.*
├── java/
│   └── ulabi_adapter.*
├── python/
│   └── ulabi_adapter.*
├── swift/
│   └── ulabi_adapter.*
├── kotlin/
│   └── ulabi_adapter.*
├── zamani/
│   └── ulabi_adapter.*
└── sankofa/
    └── ulabi_adapter.*

The important architectural point is that:

implementations/zamani/

and:

implementations/sankofa/

must be two independent adapters.

Neither should be inside the ULABI core.


---

10. Test modules

The corresponding test architecture should be:

tests/
└── interop/
    ├── type_identity/
    ├── type_mapping/
    ├── primitive_conversion/
    ├── numeric_conversion/
    ├── string_conversion/
    ├── bytes_conversion/
    ├── record_conversion/
    ├── enum_conversion/
    ├── variant_conversion/
    ├── option_conversion/
    ├── result_conversion/
    ├── collection_conversion/
    ├── ownership/
    ├── lifetime/
    ├── zero_copy/
    ├── handles/
    ├── opaque_values/
    ├── validation/
    ├── security/
    ├── compatibility/
    └── round_trip/

And:

conformance/
└── cross_language_data/
    ├── type_identity/
    ├── primitives/
    ├── records/
    ├── variants/
    ├── collections/
    ├── ownership/
    ├── lifetime/
    ├── errors/
    ├── compatibility/
    └── security/


---

11. Schemas required later

The implementation should eventually have machine-readable schemas under:

schemas/
└── interoperability/
    ├── type-id.schema.*
    ├── type-descriptor.schema.*
    ├── type-mapping.schema.*
    ├── conversion-policy.schema.*
    ├── conversion-result.schema.*
    ├── adapter-metadata.schema.*
    ├── diagnostic.schema.*
    └── compatibility-report.schema.*

These should be derived from the normative contract, not invented independently by implementation authors.


---

12. Examples required

Use:

examples/
└── interoperability/
    ├── primitive-mapping/
    ├── string-mapping/
    ├── record-mapping/
    ├── enum-mapping/
    ├── variant-mapping/
    ├── ownership-transfer/
    ├── borrowed-data/
    ├── immutable-sharing/
    ├── zero-copy/
    ├── opaque-handle/
    ├── lossless-conversion/
    ├── lossy-conversion/
    └── cross-language/

The cross-language examples should eventually demonstrate at least:

C <-> Rust
C++ <-> Python
Go <-> Java
Swift <-> Kotlin
Zamani <-> Rust
Sankofa <-> C

without making any of those languages normative.


---

13. Exact implementation order

This is the order I recommend so that you do not have to return to an earlier completed file:

Phase 1 — normative contracts

1. docs/interoperability/cross-language-data.md ← current file


2. schemas/interoperability/*


3. examples/interoperability/*


4. tests/interop/*



Phase 2 — independent implementation foundation

5. type_identity.*


6. semantic_type.*


7. type_descriptor.*


8. type_registry.*


9. conversion_policy.*


10. conversion_result.*


11. validation.*


12. diagnostics.*



Phase 3 — converters

13. type_mapping.*


14. converter.*


15. primitive_converter.*


16. numeric_converter.*


17. string_converter.*


18. bytes_converter.*


19. collection_converter.*


20. record_converter.*


21. enum_converter.*


22. variant_converter.*


23. option_converter.*


24. result_converter.*



Phase 4 — memory safety

25. ownership.*


26. lifetime.*


27. borrow.*


28. transfer.*


29. shared_value.*


30. immutable_view.*


31. allocator_boundary.*


32. zero_copy.*



Phase 5 — resources

33. handle.*


34. opaque_value.*


35. resource_lifetime.*


36. release.*


37. capability_guard.*



Phase 6 — adapter framework

38. adapter.*


39. adapter_metadata.*


40. adapter_registry.*


41. language_binding.*


42. mapping_plan.*


43. conversion_plan.*


44. compatibility_checker.*



Phase 7 — language implementations

45. C adapter


46. C++ adapter


47. Rust adapter


48. Go adapter


49. Java adapter


50. Python adapter


51. Swift adapter


52. Kotlin adapter


53. Zamani adapter


54. Sankofa adapter



Phase 8 — conformance

55. cross-language conformance vectors


56. round-trip tests


57. differential tests


58. fuzz tests


59. ownership/lifetime tests


60. security tests


61. compatibility tests




---

14. Most important architectural decision

The biggest thing I would lock down now is this:

ULABI
                    |
          Semantic Contract
                    |
       +------------+------------+
       |            |            |
     C/C++        Rust        Python
       |            |            |
    Adapter      Adapter      Adapter
       |            |            |
       +------------+------------+
                    |
              Same ULABI
              semantics

Not:

C <-> Rust
C <-> Python
Rust <-> Python
Zamani <-> Sankofa
...

That would recreate the N×N interoperability problem ULABI is supposed to solve.

Also, the repository already has the right architectural direction: ULABI-DESIGN.md establishes the universal, language-neutral contract, while the repository separately contains ABI, distributed, compatibility, and hardware specifications.

