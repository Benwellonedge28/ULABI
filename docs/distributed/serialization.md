I inspected the repository first rather than treating the proposed directory tree as authoritative.

The current repository confirms that ULABI-DESIGN.md is the architectural authority and is currently 0.2.0-draft. It already establishes the principles this document needs: Minimal Core + Standard Profiles + Extensible Ecosystem, transport independence, explicit locality, stable interface identity, compatibility, negotiation/discovery, deterministic encoding, and distributed interoperability.

I also checked the distributed area. distributed-abi.md is already a substantial distributed-boundary specification, while serialization.md, remote-calls.md, distributed-errors.md, and service-discovery.md are currently placeholders.  The compatibility family is already substantial, including forward/backward compatibility, capability discovery, feature negotiation, and graceful degradation.

So docs/distributed/serialization.md should own the serialization contract itself, not duplicate the distributed ABI, remote-call, error, or compatibility specifications.

docs/distributed/serialization.md

Copy the following into the file:

# ULABI Distributed Serialization

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/distributed/serialization.md`  
**Specification Family:** Distributed Interoperability / Serialization  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**Normative Authority:** `ULABI-DESIGN.md`, `ULABI-SPEC.md`, `ULABI-VERSIONING.md`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI serialization contract for values,
interfaces, metadata, errors, extensions, and other information that crosses
an interoperability boundary where a common in-memory representation cannot
be assumed.

Serialization provides the deterministic transformation between:

```text
ULABI Semantic Value
        |
        v
ULABI Boundary Representation
        |
        v
Serialized Representation
        |
        v
Transport / Storage
        |
        v
Serialized Representation
        |
        v
ULABI Boundary Representation
        |
        v
ULABI Semantic Value

The purpose of serialization is to preserve the semantics of a ULABI contract across independently implemented systems.

Serialization MUST NOT require participating implementations to share:

a programming language;

a compiler;

a runtime;

an operating system;

a CPU architecture;

a memory-management model;

an object layout;

a pointer representation;

a vendor;

a transport.


The serialization layer is therefore a semantic boundary, not a memory dump.


---

2. Fundamental Principle

ULABI serialization MUST serialize semantic contracts rather than implementation-specific memory layouts.

The following are fundamentally different:

Semantic Value

and:

Process Memory

A serialized representation MUST NOT depend on:

raw pointers;

process addresses;

compiler-specific object layouts;

stack layouts;

register layouts;

language-specific vtables;

implementation-specific padding;

host pointer width unless explicitly represented;

native structure layout unless explicitly standardized.


Therefore:

> A ULABI serialization MUST remain interpretable independently of the producer's internal memory representation.




---

3. Scope

This specification defines:

1. serialization terminology;


2. serialization boundaries;


3. semantic value encoding;


4. canonical encoding;


5. decoding;


6. schema identity;


7. type identity;


8. field identity;


9. version metadata;


10. optional values;


11. records;


12. enums;


13. variants;


14. lists;


15. maps;


16. strings;


17. bytes;


18. integers;


19. floating-point values;


20. handles and references;


21. errors;


22. extensions;


23. unknown fields;


24. unknown values;


25. canonical ordering;


26. length encoding;


27. nesting limits;


28. resource limits;


29. validation;


30. malformed input handling;


31. security requirements;


32. integrity boundaries;


33. streaming serialization;


34. incremental decoding;


35. zero-copy boundaries;


36. schema evolution;


37. compatibility;


38. deterministic serialization;


39. conformance requirements.



This document does NOT define:

network transports;

RPC protocols;

service discovery;

authentication protocols;

authorization policy;

consensus algorithms;

application-level schemas;

a mandatory wire protocol such as HTTP;

a mandatory existing serialization format.


ULABI MAY define or adopt one or more concrete encoding profiles later.


---

4. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



The serialization architecture is:

ULABI Semantic Model
                            |
                            v
                    Type / Schema Contract
                            |
                            v
                  Serialization Contract
                            |
             +--------------+--------------+
             |                             |
      Canonical Encoding             Extension Encoding
             |                             |
             +--------------+--------------+
                            |
                            v
                     Transport Boundary

The Distributed ABI consumes this specification.

The Distributed ABI defines the distributed execution contract.

This document defines how values cross that boundary.

Therefore this specification MUST NOT redefine distributed invocation, retry, cancellation, idempotency, locality, endpoint identity, or service discovery.

Those responsibilities belong to:

docs/distributed/distributed-abi.md

docs/distributed/remote-calls.md

docs/distributed/distributed-errors.md

docs/distributed/service-discovery.md

docs/distributed/consensus-boundaries.md



---

5. Normative Language

The following terms are normative:

MUST

MUST NOT

REQUIRED

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY

OPTIONAL


A conforming implementation MUST satisfy every applicable MUST and MUST NOT requirement.


---

6. Serialization Model

ULABI defines three conceptual layers:

Semantic Layer
      |
      v
Boundary Type Layer
      |
      v
Encoding Layer

6.1 Semantic Layer

The semantic layer describes what a value means.

Examples:

UserID
Timestamp
Money
Person
Result
Error
Capability

6.2 Boundary Type Layer

The boundary type layer describes the ULABI representation:

UInt
String
Bytes
Record
Enum
Variant
Option
Result
List
Map
Handle

6.3 Encoding Layer

The encoding layer represents the boundary type as bytes or another explicitly defined transport-neutral representation.

An implementation MUST NOT confuse these layers.


---

7. Serialization Contract

A serialization contract MUST identify:

schema identity
schema version
type identity
encoding profile
encoding version
compatibility requirements
limits
extension policy
canonicalization policy

Conceptually:

SerializationContract {
    schema_id
    schema_version
    type_id
    encoding_profile
    encoding_version
    compatibility_mode
    extension_policy
    canonicalization_policy
    limits
}

The exact binary representation is defined by the selected encoding profile.


---

8. Schema Identity

Every serialized structured value SHOULD be associated with a stable schema identity.

Conceptually:

SchemaIdentity {
    namespace
    schema_id
    major_version
}

Schema identity MUST NOT depend solely on:

source-language names;

file names;

package paths;

compiler-generated names;

memory addresses;

deployment location.


A schema identity MUST remain stable across independent implementations.


---

9. Type Identity

Every serialized type that requires independent identification MUST have a stable type identity.

Examples include:

Person
UserID
Timestamp
Result
Capability
Error

A type identity MUST NOT be inferred solely from its source-language spelling.

For example:

Rust::User

and:

Python::User

MUST NOT automatically be considered the same ULABI type.

Semantic identity MUST be established by the ULABI contract.


---

10. Field Identity

Structured records MUST use stable field identities.

Conceptually:

Field {
    field_id
    type_id
    required
    attributes
}

Field identity MUST NOT depend solely on declaration order.

For example:

Person {
    field 1 = id
    field 2 = name
}

must remain identifiable even if an implementation internally stores:

name
id

in another order.


---

11. Canonical Serialization

ULABI serialization SHOULD provide a canonical representation.

Canonical serialization is required when deterministic byte-level equality is needed.

Canonical encoding MUST define:

field ordering;

map ordering;

integer representation;

string representation;

floating-point representation;

optional-value representation;

extension ordering;

duplicate-field behavior;

length representation;

absence representation.


Two independently conforming implementations producing canonical encoding for the same semantic value MUST produce equivalent canonical output.

Where byte-for-byte identity is required by a profile, they MUST produce identical bytes.


---

12. Determinism

Canonical serialization MUST be deterministic.

Given:

Value V
Schema S
Encoding Profile E

the result MUST be:

Serialize(V, S, E) = deterministic representation

Serialization MUST NOT depend on:

hash-map iteration order;

memory addresses;

thread scheduling;

process identifiers;

random allocation;

locale;

host endianness;

implementation-specific object layout.


Unless explicitly declared by the encoding profile, serialization MUST NOT introduce nondeterministic fields.


---

13. Byte Order

An encoding profile MUST define byte order explicitly wherever multi-byte numeric representation is used.

An implementation MUST NOT assume the receiver uses the producer's native endianness.

Native host byte order MUST NOT silently become ULABI wire byte order.


---

14. Integer Serialization

ULABI integer serialization MUST define:

signedness;

width;

range;

representation;

byte order;

overflow behavior;

canonical encoding.


Examples:

Int8
Int16
Int32
Int64

UInt8
UInt16
UInt32
UInt64

Future arbitrary-width integer types MUST have explicit encoding rules.

Overflow MUST NOT silently become truncation.

A decoder MUST reject a representation that exceeds the declared type range unless the applicable type contract explicitly defines another behavior.


---

15. Boolean Serialization

Boolean values have exactly two semantic states:

true
false

The selected encoding profile MUST define their representation.

Other values MUST NOT silently become valid Boolean values.


---

16. Floating-Point Serialization

Floating-point serialization MUST define:

supported formats;

precision;

exponent range;

signed zero;

positive infinity;

negative infinity;

NaN;

NaN canonicalization;

comparison semantics where relevant.


If a profile canonicalizes NaN values, that canonicalization MUST be specified.

A decoder MUST NOT silently reinterpret an unsupported floating-point value as an unrelated numeric value.


---

17. Strings

ULABI strings are semantic Unicode strings.

UTF-8 SHOULD be the canonical string encoding unless a profile explicitly defines another representation.

The serialization contract MUST define:

encoding;

validity;

length;

maximum size;

invalid sequence behavior;

normalization policy if normalization is required.


Invalid UTF-8 MUST NOT silently become a different valid string.

Strings MUST remain distinct from arbitrary bytes.

String != Bytes


---

18. Bytes

Bytes represents arbitrary binary data.

A byte sequence:

[00 FF 7A ...]

MUST NOT be interpreted as text unless the type contract explicitly requires that interpretation.

The encoding MUST preserve byte values exactly.


---

19. Unit

Unit represents the absence of a meaningful value.

A conforming encoding profile MUST define how Unit is represented.

For example:

Result<Unit, Error>

MUST remain distinguishable from:

Result<Bytes, Error>

even if both happen to contain no payload.


---

20. Option

ULABI Option<T> has two semantic states:

None
Some(value)

The serialized representation MUST distinguish them unambiguously.

None MUST NOT be represented by an arbitrary sentinel unless the type contract explicitly defines that sentinel.


---

21. Result

ULABI Result<T,E> has two semantic states:

Success(value)
Failure(error)

The encoding MUST preserve the distinction.

An error MUST NOT be serialized as an ordinary successful value.


---

22. Lists

Lists are ordered sequences.

Conceptually:

List<T> {
    length
    elements[]
}

Serialization MUST preserve:

element order;

element count;

element type;

nesting;

maximum permitted length.


A decoder MUST enforce configured maximum lengths.


---

23. Maps

Maps MAY be supported by an encoding profile.

A map contract MUST define:

key type;

value type;

duplicate-key behavior;

ordering;

canonical ordering;

maximum entries.


If canonical ordering is required, map keys MUST have a deterministic ordering rule.

A decoder MUST NOT silently overwrite duplicate keys unless the schema explicitly defines that behavior.


---

24. Records

Records contain named or identified fields.

Conceptually:

Record {
    schema_id
    fields[]
}

Fields MUST use stable identifiers.

Record serialization MUST define:

required fields;

optional fields;

default values;

unknown-field behavior;

duplicate-field behavior;

field ordering;

maximum field count;

maximum encoded size.



---

25. Required Fields

A required field MUST be present unless the contract explicitly defines a default representation.

A decoder MUST reject a record missing a mandatory field when no valid default exists.

A newer producer MUST NOT introduce a new mandatory field when communicating with an older consumer unless compatibility has explicitly been established.


---

26. Optional Fields

Optional fields MAY be omitted.

The contract MUST distinguish:

field absent

from:

field present with zero/empty/default value

when that distinction is semantically meaningful.


---

27. Unknown Fields

Unknown fields are expected during schema evolution.

Their behavior MUST be defined by the schema.

Possible policies include:

Ignore
Preserve
Reject
RequireNegotiation

Unknown fields MUST NOT automatically be accepted.

Unknown fields MUST NOT automatically be rejected.

The schema determines the safe behavior.


---

28. Unknown Field Preservation

If an implementation acts as a transparent intermediary and the contract requires preservation, unknown fields MUST be preserved without semantic modification.

Preservation MUST retain, where applicable:

field identity;

field representation;

payload;

extension metadata;

integrity information.


An implementation MUST NOT claim transparent forwarding if it drops required unknown fields.


---

29. Duplicate Fields

The serialization contract MUST define duplicate-field behavior.

Possible policies include:

Reject
FirstWins
LastWins
Merge

The default behavior SHOULD be:

Reject

unless the schema explicitly defines another rule.

Duplicate handling MUST be deterministic.


---

30. Enums

Enums MUST use stable identifiers.

Example:

Status {
    Pending = 1
    Active = 2
    Complete = 3
}

Numeric or symbolic identifiers MUST remain stable.

An unknown enum value MUST have explicitly defined behavior.

Possible behaviors:

Preserve
MapToUnknown
Reject
Negotiate

An unknown enum value MUST NOT silently become an unrelated known value.


---

31. Variants

Variants represent tagged alternatives.

Example:

Shape =
    Circle
    Rectangle
    Triangle

Each variant MUST have a stable identity.

Unknown variants MUST have defined behavior.

A decoder MUST NOT interpret an unknown variant as a known variant solely because their payload layouts happen to resemble one another.


---

32. Handles

Handles represent resources whose implementation is outside the serialized value itself.

Examples:

FileHandle
DeviceHandle
Capability
Resource

A raw pointer MUST NOT be serialized as a portable ULABI handle.

Conceptually:

Handle {
    namespace
    resource_id
    type_id
    authority
    version
}

The actual representation is profile-specific.


---

33. References

A process-local memory address MUST NOT be serialized as a distributed reference.

For example:

0x7FFF12345678

MUST NOT be assumed meaningful on another process or host.

Distributed references MUST use an explicitly defined semantic representation.


---

34. Ownership

Serialization MUST preserve the ownership contract associated with a value.

Possible ownership semantics include:

Borrow
Copy
Transfer
Move
Shared
Lease
Reference

Serialization MUST NOT silently convert:

Borrow

into:

Transfer

or:

Shared

into:

Owned

when doing so changes program semantics.


---

35. Lifetime

A serialized value MUST NOT implicitly extend or terminate a resource lifetime unless the contract explicitly defines that behavior.

For resource-bearing values, the serialization contract MUST identify whether the serialized representation contains:

a snapshot;

a reference;

a lease;

a capability;

a transferable resource;

an opaque token.



---

36. Capability Serialization

Capabilities are security-sensitive.

A capability MUST NOT become usable merely because its serialized bytes can be decoded.

Serialization of a capability MUST preserve:

capability identity;

authority;

scope;

restrictions;

expiration;

integrity;

issuer or authority information where required.


Decoding MUST NOT grant additional authority.


---

37. Security-Sensitive Values

Security-sensitive serialized values MUST be explicitly identified.

Examples:

authentication credentials;

authorization tokens;

capabilities;

cryptographic keys;

integrity metadata;

security policies.


Unknown security-sensitive fields MUST fail closed where their absence could weaken security.


---

38. Extensions

ULABI supports explicit extension points.

Conceptually:

Extension {
    extension_id
    version
    flags
    length
    payload
}

Unknown extensions MUST have a defined policy.

Possible policies:

Ignore
Preserve
Reject
RequireNegotiation

An extension MUST NOT silently redefine Core semantics.


---

39. Extension Isolation

Extensions MUST remain isolated.

For example:

ULABI Core
    |
    +-- Security Extension
    |
    +-- Streaming Extension
    |
    +-- Tensor Extension
    |
    +-- Hardware Extension
    |
    +-- Distributed Extension

The presence of one extension MUST NOT silently imply another extension.

Dependencies MUST be explicitly declared.


---

40. Versioning

Serialized representations MUST be version-aware where evolution requires it.

The implementation MUST be able to distinguish:

KnownVersion
SupportedVersion
UnsupportedVersion
UnknownVersion
CompatibleVersion
IncompatibleVersion

Version semantics are governed by:

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

This document defines serialization consequences of those version rules.


---

41. Schema Evolution

Schema evolution SHOULD be additive.

Safe additions may include:

optional fields;

optional extensions;

new diagnostics;

new capabilities;

new variants with defined unknown behavior;

new enum values with defined unknown behavior.


Existing field identifiers MUST retain their meanings.

Existing type identifiers MUST retain their meanings.

Existing encodings MUST NOT silently acquire incompatible meanings.


---

42. Breaking Schema Changes

A breaking schema change MUST be detectable.

Examples include changing:

UInt32

to:

String

without a compatible migration rule.

Other breaking changes include:

changing signedness;

changing ownership;

changing lifetime;

changing requiredness;

changing semantic meaning;

changing security requirements;

changing encoding interpretation.


A breaking change MUST use an explicit incompatible version, schema identity, profile, or negotiated migration mechanism.


---

43. Canonical Field Ordering

When canonical encoding is required, field ordering MUST be deterministic.

The ordering mechanism MUST be explicitly specified.

Possible mechanisms include:

field_id ascending

or another formally defined canonical order.

Implementations MUST NOT use declaration order unless declaration order is itself the normative canonical rule.


---

44. Length Encoding

Every variable-length structure MUST have an explicitly defined length or termination rule.

This includes:

strings;

bytes;

lists;

maps;

records;

extension payloads;

nested containers.


Length encoding MUST be bounded.

A decoder MUST reject impossible or excessive lengths before allocating unbounded memory.


---

45. Resource Limits

A conforming decoder MUST support resource limits.

At minimum, implementations SHOULD be able to limit:

maximum_message_size
maximum_field_count
maximum_string_size
maximum_bytes_size
maximum_list_length
maximum_map_entries
maximum_nesting_depth
maximum_extension_size
maximum_total_allocation
maximum_decoding_time

Limits MUST be applied before unsafe resource consumption occurs.


---

46. Integer Overflow During Decoding

Length fields and counters are security-sensitive.

A decoder MUST detect:

integer overflow;

integer underflow;

multiplication overflow;

addition overflow;

invalid length conversion.


For example:

count * element_size

MUST be checked before allocation.

Arithmetic overflow MUST NOT result in undersized allocation followed by buffer corruption.


---

47. Nesting Limits

Nested values MUST be bounded.

For example:

List<List<List<...>>>

MUST NOT allow unbounded recursive nesting.

The decoder MUST enforce a maximum nesting depth.

The limit MAY be profile-specific.


---

48. Malformed Input

Malformed serialized input MUST NOT cause:

memory corruption;

undefined behavior;

arbitrary code execution;

capability escalation;

unbounded resource consumption;

uncontrolled recursion;

process-wide termination unless explicitly required by a safety profile.


Malformed input SHOULD produce a structured decoding error.


---

49. Decoder State

A decoder MUST maintain explicit validation state.

Conceptually:

Receive
   |
Validate Envelope
   |
Validate Version
   |
Validate Schema
   |
Validate Type
   |
Validate Lengths
   |
Validate Structure
   |
Validate Security Metadata
   |
Decode
   |
Validate Semantic Invariants
   |
Produce Value

A value MUST NOT become observable to application code before mandatory validation is complete.


---

50. Decode-Then-Validate Rule

Where semantic validation is separate from structural decoding, structural decoding MUST complete safely before the value is exposed.

However, implementations MUST NOT treat successful byte decoding as proof that a value is semantically valid.

For example:

Decoded UInt = 999

may still violate:

Age <= 150

if the schema establishes that invariant.


---

51. Semantic Validation

Schemas MAY define semantic constraints such as:

range
pattern
length
uniqueness
cross-field dependency
required relationship
security policy
resource policy

The validator MUST enforce all mandatory constraints.

Unknown semantic constraints that are required for safe interpretation MUST cause rejection.


---

52. Authentication and Integrity Boundary

Serialization itself does not imply authentication or integrity.

A serialized message MAY require an external integrity mechanism.

Where integrity metadata is part of the contract, the implementation MUST verify it before trusting the protected data.

Serialization MUST NOT claim that canonical encoding provides authenticity.


---

53. Confidentiality

Serialization does not inherently provide confidentiality.

Sensitive values MAY require encryption through an appropriate security or transport profile.

An implementation MUST NOT claim that serialization alone protects secrets from observers.


---

54. Compression

Compression MAY be applied outside the semantic serialization layer.

If compression is part of a serialization profile, the profile MUST define:

algorithm identity;

version;

limits;

decompression bounds;

failure behavior.


Compressed data MUST NOT bypass message-size or allocation limits.


---

55. Streaming Serialization

Large values MAY be serialized as streams.

Streaming MUST preserve semantic boundaries.

For example:

Stream<Bytes>

is not automatically equivalent to:

Bytes

A streaming profile MUST define:

chunk boundaries;

sequence numbering where required;

completion;

cancellation;

errors;

backpressure;

maximum chunk size;

maximum total size.



---

56. Incremental Decoding

An implementation MAY decode incrementally.

Incremental decoding MUST NOT expose incomplete semantic values as complete values.

Conceptually:

Chunk 1
   |
Chunk 2
   |
Chunk 3
   |
Complete
   |
Validate
   |
Expose Value

An incomplete stream MUST remain distinguishable from a valid empty value.


---

57. Partial Serialization

A serializer MUST NOT silently emit a partial value as a complete value.

If serialization fails after producing partial output, the result MUST be marked incomplete or invalid according to the selected profile.

Consumers MUST NOT assume partial bytes represent a valid ULABI value.


---

58. Zero-Copy

Zero-copy is an optimization, not a semantic requirement.

A zero-copy implementation MAY use:

shared memory;

memory mapping;

immutable buffers;

hardware-supported buffers.


However, zero-copy MUST preserve:

ownership;

lifetime;

access permissions;

synchronization;

memory visibility;

integrity;

security.


A serialized representation MUST remain portable even when zero-copy is unavailable, unless the profile explicitly requires a zero-copy environment.


---

59. Native Object Layout

ULABI serialization MUST NOT depend on native object layout.

The following MUST NOT be assumed portable:

struct padding
vtable pointers
pointer size
pointer alignment
compiler ABI
native enum layout
native bool layout
native union layout

An implementation MAY optimize internally, but the boundary representation must remain contract-defined.


---

60. Schema Registry Integration

A schema registry MAY provide schemas by identity.

A registry lookup MUST NOT be required merely to determine basic structural safety when the message contains sufficient schema information.

If a registry is used, the contract SHOULD define:

schema_id
schema_version
registry_namespace
integrity
trust requirements

A registry MUST NOT be treated as authoritative merely because it is reachable.


---

61. Schema Caching

Implementations MAY cache schemas.

Cached schemas MUST be associated with their exact identity and version.

An implementation MUST NOT reuse a cached schema merely because two schemas have similar names.

Cache invalidation MUST preserve version correctness.


---

62. Unknown Schema

When a schema cannot be resolved, the implementation MUST classify the condition explicitly.

Possible outcomes:

SchemaAvailable
SchemaUnavailable
SchemaUnsupported
SchemaInvalid
SchemaIncompatible

The implementation MUST NOT guess the schema.


---

63. Compatibility

Serialization compatibility MUST be determined by the complete contract.

Two representations are compatible only when the applicable rules agree on:

semantic type;

encoding;

required fields;

optional fields;

ownership;

lifetime;

security;

limits;

extension behavior;

version;

canonicalization where required.


Byte-level similarity is insufficient evidence of compatibility.


---

64. Forward Compatibility

Forward compatibility is governed by:

docs/compatibility/forwards-compatibility.md

This serialization specification requires:

explicit unknown-field policy;

explicit unknown-extension policy;

explicit unknown-enum policy;

explicit unknown-variant policy;

explicit version handling;

bounded decoding;

security-safe rejection.


An implementation MUST NOT claim forward compatibility merely because it ignores unknown bytes.


---

65. Backward Compatibility

Backward compatibility is governed by:

docs/compatibility/backwards-compatibility.md

Serialization SHOULD prefer additive changes that preserve existing representations.

Breaking changes MUST be detectable.


---

66. Graceful Degradation

When a newer representation contains optional functionality unsupported by the receiver, graceful degradation MAY occur when explicitly permitted.

For example:

New Feature
     |
Supported?
  /     \
YES      NO
 |        |
Use     Fallback

Graceful degradation MUST NOT remove required security, ownership, integrity, or correctness guarantees.

The detailed degradation policy belongs to:

docs/compatibility/graceful-degradation.md


---

67. Feature Negotiation

Serialization MAY be selected or constrained through feature negotiation.

The negotiation mechanism belongs to:

docs/compatibility/feature-negotiation.md

Serialization MUST expose enough metadata for the negotiated encoding profile to be unambiguous.

An implementation MUST NOT silently switch to an incompatible encoding.


---

68. Capability Discovery

Capability discovery belongs to:

docs/compatibility/capability-discovery.md

Serialization MAY carry capability metadata, but decoding a capability MUST NOT itself grant that capability.


---

69. Distributed ABI Integration

The Distributed ABI uses serialization for values crossing distributed boundaries.

The relationship is:

Distributed ABI
      |
      +-- Remote Invocation
      |
      +-- Serialization
      |
      +-- Distributed Errors
      |
      +-- Service Discovery
      |
      +-- Consensus Boundaries

distributed-abi.md defines the distributed execution semantics.

This document defines value representation.

Neither document should duplicate the other.


---

70. Remote Call Integration

Remote invocation metadata MAY itself be serialized.

The remote-call contract defines:

invocation identity;

deadlines;

cancellation;

idempotency;

retries;

delivery semantics.


This document defines the representation of those values when serialized.

The remote-call specification remains authoritative for their meaning.


---

71. Distributed Error Integration

Errors crossing a boundary MUST have a serializable representation.

The distributed error specification defines:

error classification;

retryability;

remote failure semantics;

partial failure;

unknown outcome.


This document defines how an error value is encoded.

An unknown error MUST remain distinguishable from success.


---

72. Consensus Integration

Serialization used by consensus or replicated-state mechanisms has stronger requirements.

Consensus-related serialized data SHOULD use canonical deterministic encoding.

All participants MUST agree on:

schema;

version;

encoding;

field ordering;

numeric representation;

extension handling.


Consensus MUST NOT depend on implementation-specific serialization.

Detailed consensus semantics belong to:

docs/distributed/consensus-boundaries.md


---

73. Cryptographic Integration

Canonical serialization MAY be used as input to:

hashes;

signatures;

MACs;

content identifiers;

Merkle structures.


When serialized bytes are authenticated or signed, the exact canonicalization rules MUST be fixed.

A signature MUST NOT cover an ambiguous representation.

Conceptually:

Semantic Value
      |
Canonical Serialization
      |
Hash / Signature


---

74. Replay Considerations

Serialization alone does not prevent replay.

If a protocol requires replay protection, the surrounding contract MUST define appropriate metadata such as:

nonce;

sequence;

timestamp;

expiration;

invocation identity;

session identity.


Serialization MUST preserve such metadata exactly when it is part of the security contract.


---

75. Resource Exhaustion

Decoders MUST defend against hostile representations designed to consume:

memory;

CPU;

stack;

recursion depth;

file descriptors;

handles;

network bandwidth.


Examples include:

huge declared length
deep nesting
many duplicate fields
many tiny extensions
pathological maps
large integer values
recursive references

Limits MUST be enforced before dangerous work is performed.


---

76. Recursive Values

If recursive types are supported, the encoding profile MUST define:

reference representation;

termination;

maximum depth;

cycle behavior;

identity semantics.


A recursive value MUST NOT cause unbounded decoding.


---

77. Cycles

If the type system permits cyclic object graphs, serialization MUST explicitly define whether cycles are:

Forbidden
DetectedAndRejected
ReferenceEncoded
IdentityPreserved

An implementation MUST NOT accidentally recurse forever.


---

78. Aliasing

If two fields refer to the same semantic object, the serialization contract MUST define whether aliasing is:

Preserved
Collapsed
Copied
Forbidden

Serialization MUST NOT silently change aliasing semantics when those semantics are observable.


---

79. Equality

A serialization profile MUST distinguish:

semantic equality

from:

byte equality

Two values MAY be semantically equal while having different non-canonical representations.

Canonical profiles MAY require:

semantic equality
        =>
canonical byte equality

where explicitly defined.


---

80. Hashing

When serialized bytes are used for hashing, the profile MUST specify whether the hash covers:

canonical representation;

schema identity;

version;

extension metadata;

security metadata.


Hashing an implementation-specific representation MUST NOT be presented as a ULABI semantic hash.


---

81. Error Model

Serialization errors SHOULD be structured.

At minimum, implementations SHOULD distinguish:

InvalidEncoding
TruncatedInput
UnsupportedEncoding
UnsupportedVersion
UnknownSchema
UnknownType
InvalidType
InvalidField
MissingRequiredField
DuplicateField
InvalidValue
LimitExceeded
InvalidExtension
IntegrityFailure
SecurityViolation

Implementations MAY define additional errors.

Errors MUST NOT expose sensitive input unnecessarily.


---

82. Failure Atomicity

Where practical, failed decoding SHOULD leave the destination object unchanged.

An implementation MUST NOT expose partially decoded state as a valid complete value.

For mutable destination objects, transactional decoding is RECOMMENDED where the security or correctness profile requires it.


---

83. Unknown Data Policy

Every extensible serialization contract MUST define an unknown-data policy.

The policy MUST be one or more of:

Ignore
Preserve
Reject
Negotiate

Security-critical unknown information MUST NOT be treated as safely ignorable unless the governing security contract explicitly establishes that rule.


---

84. Security Requirements

A conforming implementation MUST:

1. validate lengths;


2. detect arithmetic overflow;


3. enforce nesting limits;


4. enforce allocation limits;


5. reject malformed encodings;


6. prevent capability escalation;


7. prevent pointer injection;


8. avoid uncontrolled recursion;


9. distinguish unknown from valid;


10. preserve required integrity metadata;


11. fail closed for unsupported mandatory security semantics.



A serializer MUST NOT emit malformed representations intentionally.


---

85. Implementation Independence

A conforming implementation MAY be written in any programming language.

Examples include:

C
C++
Rust
Go
Java
Python
Swift
Kotlin
Fortran
Ada
Zamani
Sankofa

ULABI serialization MUST NOT require any of these languages.

An implementation MAY use any internal object model.

Only the boundary contract is standardized.


---

86. Conformance Requirements

A serialization implementation is conforming only if it passes the applicable serialization conformance tests.

At minimum tests MUST cover:

primitive values;

signed integers;

unsigned integers;

Boolean;

strings;

bytes;

lists;

records;

optional fields;

enums;

variants;

Result;

unknown fields;

duplicate fields;

unknown extensions;

malformed input;

length overflow;

nesting limits;

canonical encoding;

schema mismatch;

version mismatch;

security-sensitive metadata.


Distributed profiles MUST additionally test:

remote value transfer;

distributed references;

ownership;

resource limits;

streaming;

partial data;

error serialization;

compatibility.



---

87. Required Conformance Properties

A conforming serializer MUST satisfy:

Determinism
Type Preservation
Semantic Preservation
Boundary Isolation
Bounded Decoding
Version Awareness
Schema Awareness
Security Preservation
Error Determinism
Implementation Independence


---

88. Reference Test Vectors

ULABI SHOULD maintain normative serialization test vectors.

Each test vector SHOULD contain:

vector_id
schema_id
schema_version
type_id
semantic_value
encoded_representation
canonical_flag
expected_decode_result
expected_error

Test vectors MUST be independent of a particular programming language.


---

89. Cross-Implementation Testing

At least two independently implemented serializers SHOULD be tested against the same normative vectors.

Example:

Implementation A
      |
      +----> Test Vectors
      |
Implementation B
      |
      +----> Test Vectors

The goal is to prevent a single implementation from becoming the de facto definition of the standard.


---

90. Fuzz Testing

Decoders SHOULD be fuzz tested.

Fuzzing MUST include:

random bytes;

truncated values;

malformed lengths;

malformed UTF-8;

invalid tags;

duplicate fields;

deeply nested values;

oversized values;

unknown extensions;

corrupted canonical representations.


A fuzz failure MUST NOT result in:

memory corruption;

undefined behavior;

uncontrolled resource consumption;

privilege escalation.



---

91. Differential Testing

Independent implementations SHOULD perform differential testing.

For equivalent semantic values:

Implementation A
        |
        v
 Serialization A
        |
        v
Implementation B
        |
        v
 Semantic Value

The resulting semantic value MUST remain equivalent under the applicable contract.


---

92. Compatibility Testing

The conformance suite MUST test:

Old Encoder -> New Decoder
New Encoder -> Old Decoder

where the contracts claim compatibility.

It MUST also test safe rejection where compatibility is not claimed.


---

93. Determinism Testing

Canonical serializers MUST be tested repeatedly.

Given identical:

schema
version
value
encoding profile

the output MUST remain identical.

Tests SHOULD repeat serialization across:

processes;

machines;

architectures;

implementations;

execution orders.



---

94. Reference Implementation Boundary

ULABI MAY provide reference implementations.

A reference implementation MUST NOT become the normative definition.

The normative definition is the specification.

Reference implementations exist to demonstrate and validate the contract.


---

95. Integration Contract

This document integrates with:

Core ABI

docs/abi/core-abi.md

Provides the fundamental ABI contract.

Data Types

docs/abi/data-types.md

Defines the semantic type system consumed by serialization.

Memory Model

docs/abi/memory-model.md

Defines ownership and memory-boundary semantics that serialization MUST respect.

Distributed ABI

docs/distributed/distributed-abi.md

Defines distributed execution semantics.

Remote Calls

docs/distributed/remote-calls.md

Defines invocation semantics.

Distributed Errors

docs/distributed/distributed-errors.md

Defines distributed failure semantics.

Service Discovery

docs/distributed/service-discovery.md

Defines endpoint discovery.

Consensus Boundaries

docs/distributed/consensus-boundaries.md

Defines consensus-related requirements.

Forward Compatibility

docs/compatibility/forwards-compatibility.md

Defines forward-evolution semantics.

Backward Compatibility

docs/compatibility/backwards-compatibility.md

Defines backward-evolution semantics.

Feature Negotiation

docs/compatibility/feature-negotiation.md

Defines feature selection.

Capability Discovery

docs/compatibility/capability-discovery.md

Defines capability discovery.

Graceful Degradation

docs/compatibility/graceful-degradation.md

Defines safe fallback behavior.

Standards

docs/standards/conformance.md

Defines conformance requirements.

docs/standards/test-suite.md

Defines the shared test infrastructure.


---

96. No Duplicate Ownership

The following ownership boundaries are mandatory:

Serialization
    -> value representation

Distributed ABI
    -> distributed execution semantics

Remote Calls
    -> invocation semantics

Distributed Errors
    -> distributed failure semantics

Service Discovery
    -> endpoint discovery

Consensus Boundaries
    -> agreement/consensus semantics

Forward Compatibility
    -> future-version compatibility

Backward Compatibility
    -> older-contract compatibility

Feature Negotiation
    -> selecting supported features

Capability Discovery
    -> discovering capabilities

Graceful Degradation
    -> safe fallback behavior

No document should redefine another document's normative contract.


---

97. Required Serialization Modules

The eventual implementation SHOULD be decomposed into language-neutral modules with equivalent implementations in each supported language.

Required conceptual modules:

serialization/
├── codec
├── encoder
├── decoder
├── canonicalizer
├── schema
├── schema_registry
├── type_registry
├── value
├── primitive_codec
├── record_codec
├── enum_codec
├── variant_codec
├── collection_codec
├── option_codec
├── result_codec
├── handle_codec
├── reference_codec
├── extension_codec
├── version_codec
├── validation
├── limits
├── errors
├── security
├── streaming
├── compatibility
└── test_vectors

These are conceptual module names, not language-specific implementation requirements.


---

98. Required Code Responsibilities

codec

Coordinates encoding and decoding.

Must not contain language-specific semantics.

encoder

Transforms validated semantic values into the selected encoding.

decoder

Transforms encoded representations into validated semantic values.

canonicalizer

Produces canonical representations when required.

schema

Represents schema identity and schema metadata.

schema_registry

Resolves schemas by stable identity.

type_registry

Resolves ULABI type identities.

value

Provides language-neutral semantic value abstractions.

primitive_codec

Encodes primitive ULABI types.

record_codec

Encodes structured records.

enum_codec

Encodes enumerations.

variant_codec

Encodes tagged variants.

collection_codec

Encodes lists and maps.

option_codec

Encodes Option<T>.

result_codec

Encodes Result<T,E>.

handle_codec

Encodes semantic resource handles.

reference_codec

Encodes distributed references.

extension_codec

Encodes and validates extension blocks.

version_codec

Handles serialization-version metadata.

validation

Performs structural and semantic validation.

limits

Enforces resource limits.

errors

Provides structured serialization errors.

security

Handles security-sensitive serialization rules without granting authority.

streaming

Provides incremental encoding/decoding.

compatibility

Evaluates serialization compatibility without replacing the broader ULABI compatibility layer.

test_vectors

Loads and validates normative serialization vectors.


---

99. Required Schemas

The repository SHOULD eventually contain schemas for:

schemas/
├── serialization/
│   ├── envelope
│   ├── schema
│   ├── type
│   ├── field
│   ├── extension
│   ├── error
│   ├── version
│   └── limits

The exact schema language is intentionally implementation-independent.


---

100. Required Test Families

The repository SHOULD eventually contain:

tests/
├── serialization/
│   ├── primitives
│   ├── integers
│   ├── floating_point
│   ├── strings
│   ├── bytes
│   ├── records
│   ├── enums
│   ├── variants
│   ├── option
│   ├── result
│   ├── maps
│   ├── lists
│   ├── handles
│   ├── references
│   ├── extensions
│   ├── canonicalization
│   ├── compatibility
│   ├── malformed_input
│   ├── limits
│   ├── security
│   ├── streaming
│   └── differential


---

101. Required Conformance Tests

The conformance suite SHOULD define:

ULABI-SER-001  Boolean encoding
ULABI-SER-002  Integer encoding
ULABI-SER-003  String encoding
ULABI-SER-004  Byte encoding
ULABI-SER-005  List encoding
ULABI-SER-006  Map encoding
ULABI-SER-007  Record encoding
ULABI-SER-008  Enum encoding
ULABI-SER-009  Variant encoding
ULABI-SER-010  Option encoding
ULABI-SER-011  Result encoding
ULABI-SER-012  Unknown field handling
ULABI-SER-013  Unknown extension handling
ULABI-SER-014  Duplicate field handling
ULABI-SER-015  Canonical encoding
ULABI-SER-016  Version handling
ULABI-SER-017  Schema mismatch
ULABI-SER-018  Malformed input
ULABI-SER-019  Length overflow
ULABI-SER-020  Nesting limits
ULABI-SER-021  Resource limits
ULABI-SER-022  Security metadata
ULABI-SER-023  Streaming
ULABI-SER-024  Cross-implementation decoding
ULABI-SER-025  Differential serialization


---

102. Conformance Invariant

A conforming implementation MUST satisfy:

Serialize(valid value)
        |
        v
valid representation
        |
        v
Decode
        |
        v
semantically equivalent value

subject to the applicable schema, profile, version, ownership, security, and compatibility rules.


---

103. Final Architectural Rule

ULABI serialization is not:

memory dump

It is:

semantic contract
      +
type identity
      +
schema identity
      +
version
      +
deterministic representation
      +
validation
      +
security

The central rule is:

> ULABI serializes meaning, not implementation.



A C implementation, Rust implementation, Python implementation, Java implementation, Zamani implementation, or Sankofa implementation may use completely different internal representations while producing interoperable ULABI boundary representations.

ULABI therefore remains independent of every programming language, compiler, runtime, operating system, processor architecture, company, and project.

## What should be done under which files

The important thing is **not to immediately edit all of these files**. We should establish ownership in dependency order so that later files consume earlier contracts rather than forcing earlier files to be rewritten.

### Phase 1 — independent serialization foundation

These are the files/modules that can be completed with the fewest dependencies:

| Order | File | What it owns |
|---|---|---|
| 1 | `docs/distributed/serialization.md` | **The complete serialization boundary contract** |
| 2 | `schemas/serialization/envelope.*` | Serialization envelope structure |
| 3 | `schemas/serialization/schema.*` | Schema identity/version structure |
| 4 | `schemas/serialization/type.*` | Type identity |
| 5 | `schemas/serialization/field.*` | Stable field identity |
| 6 | `schemas/serialization/extension.*` | Extension structure |
| 7 | `schemas/serialization/limits.*` | Serialization resource limits |
| 8 | `schemas/serialization/error.*` | Serialization error structure |

The exact schema extension (`.json`, `.yaml`, `.toml`, etc.) should be determined by the repository's eventual schema tooling rather than inventing a language-specific format now.

### Phase 2 — serialization implementation modules

The implementation should then be split into:

```text
serialization/
├── codec
├── encoder
├── decoder
├── canonicalizer
├── schema
├── schema_registry
├── type_registry
├── value
├── primitive_codec
├── record_codec
├── enum_codec
├── variant_codec
├── collection_codec
├── option_codec
├── result_codec
├── handle_codec
├── reference_codec
├── extension_codec
├── version_codec
├── validation
├── limits
├── errors
├── security
├── streaming
└── compatibility

The critical dependency direction should be:

value
  ↓
type/schema
  ↓
primitive/record/enum/variant codecs
  ↓
encoder / decoder
  ↓
canonicalizer
  ↓
validation + limits
  ↓
serialization codec

This prevents the encoder and decoder from independently inventing type semantics.

Phase 3 — compatibility integration

These existing specifications already have separate ownership:

docs/compatibility/
├── backwards-compatibility.md
├── forwards-compatibility.md
├── feature-negotiation.md
├── capability-discovery.md
└── graceful-degradation.md

The serialization specification should consume them, not redefine them. The repository already has these documents, and the forward-compatibility specification explicitly separates its responsibility from complete encoding rules.

Phase 4 — distributed integration

Once serialization is stable:

docs/distributed/
├── distributed-abi.md
├── serialization.md          ← this document
├── remote-calls.md
├── service-discovery.md
├── distributed-errors.md
└── consensus-boundaries.md

Their responsibilities should remain:

distributed-abi.md
    ↓
distributed execution contract

serialization.md
    ↓
data representation contract

remote-calls.md
    ↓
invocation contract

distributed-errors.md
    ↓
distributed failure contract

service-discovery.md
    ↓
endpoint discovery contract

consensus-boundaries.md
    ↓
agreement/consensus boundary

That separation is particularly important because the existing distributed-abi.md already establishes serialization as a separate dependency rather than defining a second serialization system.

Phase 5 — tests

Then create:

tests/serialization/
├── primitives/
├── integers/
├── floating_point/
├── strings/
├── bytes/
├── records/
├── enums/
├── variants/
├── option/
├── result/
├── lists/
├── maps/
├── handles/
├── references/
├── extensions/
├── canonicalization/
├── compatibility/
├── malformed_input/
├── limits/
├── security/
├── streaming/
└── differential/

And:

conformance/serialization/
├── ULABI-SER-001
├── ULABI-SER-002
...
└── ULABI-SER-025

The test suite should consume the specification, not define it.

Phase 6 — reference implementation

Finally:

reference/
└── serialization/
    ├── encoder
    ├── decoder
    ├── validator
    ├── canonicalizer
    └── test_vectors

The reference implementation demonstrates the specification but must not become the specification.


---

Complete required distributed code-module map

Once the whole distributed subsystem is implemented, the code responsibilities should look like this:

distributed/
│
├── abi
├── invocation
├── serialization
│   ├── codec
│   ├── encoder
│   ├── decoder
│   ├── canonicalizer
│   ├── schema
│   ├── schema_registry
│   ├── type_registry
│   ├── value
│   ├── primitive_codec
│   ├── record_codec
│   ├── enum_codec
│   ├── variant_codec
│   ├── collection_codec
│   ├── option_codec
│   ├── result_codec
│   ├── handle_codec
│   ├── reference_codec
│   ├── extension_codec
│   ├── version_codec
│   ├── validation
│   ├── limits
│   ├── errors
│   ├── security
│   ├── streaming
│   └── compatibility
│
├── errors
├── service_discovery
├── endpoint
├── transport_adapter
├── capability
├── negotiation
├── consistency
├── ordering
├── retry
├── cancellation
├── deadlines
├── resource_limits
└── observability

These are conceptual modules, deliberately not tied to Rust, C++, Sankofa, Zamani, or any other implementation language.

One important architectural decision

I would lock this rule into ULABI now:

> Serialization is a foundational boundary service, but it is not the distributed ABI itself.



That prevents a common architectural failure where serialization starts absorbing remote calls, retries, service discovery, consensus, security, and compatibility until it becomes an unmaintainable “universal protocol.”

The current repository architecture already points in the right direction: ULABI-DESIGN.md establishes transport-independent distributed interoperability, while the distributed ABI separates serialization from invocation and other distributed concerns.

This version of docs/distributed/serialization.md therefore gives the file its complete ownership boundary and integration contract up front, so the subsequent implementation files can be built against it without requiring us to redesign this document merely because another module is added.