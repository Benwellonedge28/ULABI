I read the current ULABI-DESIGN.md first and then inspected the existing architecture/compatibility documentation so this specification does not duplicate material already established elsewhere. The master design is currently 0.2.0-draft and establishes Minimal Core + Standard Profiles + Extensible Ecosystem, explicit semantics, stable interfaces, compatibility, language/runtime/platform neutrality, and implementation independence.

I also checked the current compatibility directory. The target docs/compatibility/backwards-compatibility.md is still only a placeholder, while the related capability-discovery, feature-negotiation, forwards-compatibility, and graceful-degradation documents also exist as separate files.

So backwards-compatibility.md should own the rules for preserving older contracts, while negotiation, forward compatibility, and graceful degradation will own their respective mechanisms. That prevents the documents from becoming repetitive.

1. docs/compatibility/backwards-compatibility.md

Copy the following entire document into:

docs/compatibility/backwards-compatibility.md

# ULABI Backward Compatibility

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/compatibility/backwards-compatibility.md`  
**Specification Family:** Compatibility  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**Normative Authority:** `ULABI-DESIGN.md`, `ULABI-SPEC.md`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the requirements for backward compatibility in ULABI.

Backward compatibility means that an implementation supporting a newer
ULABI contract can continue to correctly interact with components that were
valid under an earlier compatible contract.

Backward compatibility is a fundamental ULABI property.

A ULABI implementation MUST NOT claim backward compatibility merely because
old interface names remain present.

Compatibility MUST be evaluated against the complete observable contract.

The contract includes, where applicable:

- interface identity;
- interface version;
- function identity;
- function signature;
- parameter types;
- return types;
- type identity;
- type representation;
- field identity;
- field representation;
- enum identity;
- variant identity;
- error identity;
- calling convention;
- alignment requirements;
- ownership;
- lifetime;
- mutability;
- effects;
- capabilities;
- execution semantics;
- locality;
- cancellation;
- resource requirements;
- encoding;
- validation rules;
- security requirements;
- profile dependencies.

---

# 2. Architectural Authority

This document refines the compatibility principles established by
`ULABI-DESIGN.md`.

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.

Backward compatibility MUST preserve the semantics of the existing Core.

Extensions MUST NOT redefine existing Core behavior.

This document does not define:

- forward compatibility;
- feature negotiation;
- capability discovery;
- graceful degradation;
- complete versioning policy;
- certification procedures.

Those concerns are defined by their respective specifications.

Related specifications include:

```text
ULABI-DESIGN.md
    |
    +-- ULABI-SPEC.md
    |
    +-- ULABI-VERSIONING.md
    |
    +-- docs/compatibility/backwards-compatibility.md
    |
    +-- docs/compatibility/forwards-compatibility.md
    |
    +-- docs/compatibility/feature-negotiation.md
    |
    +-- docs/compatibility/capability-discovery.md
    |
    +-- docs/compatibility/graceful-degradation.md


---

3. Normative Language

The following terms are normative:

MUST — absolute requirement.

MUST NOT — absolute prohibition.

SHALL — equivalent to MUST.

SHALL NOT — equivalent to MUST NOT.

SHOULD — strong recommendation.

SHOULD NOT — strong recommendation against.

MAY — optional behavior.


A conforming implementation MUST satisfy all applicable MUST and MUST NOT requirements.


---

4. Fundamental Compatibility Principle

ULABI compatibility is semantic.

Therefore:

Same Name
    !=
Compatible Interface

and:

Different Internal Representation
    !=
Incompatibility

Two implementations are backward compatible when the newer implementation continues to honor the observable contract required by the older contract.

Internal implementation changes are permitted when they do not violate the ULABI boundary contract.


---

5. Compatibility Is Contract-Based

Compatibility MUST be evaluated against the complete interface contract.

The minimum conceptual model is:

ULABI Interface
      |
      +-- Identity
      |
      +-- Version
      |
      +-- Types
      |
      +-- Functions
      |
      +-- Errors
      |
      +-- Encoding
      |
      +-- Calling Convention
      |
      +-- Ownership
      |
      +-- Lifetime
      |
      +-- Effects
      |
      +-- Capabilities
      |
      +-- Execution Semantics
      |
      +-- Security
      |
      +-- Profile Dependencies

Changing any of these may affect compatibility.

A compatibility implementation MUST therefore examine the relevant contract dimensions rather than only source-level declarations.


---

6. Compatibility Classes

ULABI defines four primary compatibility outcomes:

Compatible
Conditionally Compatible
Incompatible
Unknown

6.1 Compatible

The newer implementation preserves all required semantics of the older contract.

The old consumer may continue using the interface without modification.


---

6.2 Conditionally Compatible

Compatibility exists only under explicitly stated conditions.

Examples:

a profile must be present;

a feature must be negotiated;

a capability must be available;

a particular data range must be respected;

a transport constraint must be satisfied.


Conditions MUST be machine-detectable or explicitly documented.


---

6.3 Incompatible

The newer implementation cannot safely preserve the old contract.

Examples include:

changed function semantics;

incompatible parameter representation;

incompatible ownership;

incompatible lifetime;

removed mandatory functionality;

changed error semantics;

incompatible calling convention.


The implementation MUST NOT falsely advertise compatibility.


---

6.4 Unknown

Compatibility cannot be established with sufficient evidence.

An implementation MUST NOT treat Unknown as Compatible.

Security-sensitive systems SHOULD treat Unknown as incompatible until compatibility is established.


---

7. Compatibility Levels

Compatibility SHALL be evaluated at multiple levels.

Level 0
Interface Identity

Level 1
Type Compatibility

Level 2
Function Compatibility

Level 3
Binary / Calling Compatibility

Level 4
Semantic Compatibility

Level 5
Security / Capability Compatibility

Level 6
Profile Compatibility

A higher compatibility level MUST NOT be claimed unless all lower applicable levels have been satisfied.


---

8. Interface Identity Compatibility

An interface MUST have a stable identity.

The identity MUST NOT depend solely on:

file names;

source-language names;

memory addresses;

compiler-generated symbols;

implementation-specific names.


An interface identifier MUST remain stable across compatible revisions.

Changing an interface identifier normally creates a new interface identity.


---

9. Function Identity Compatibility

Function identity MUST remain stable across compatible revisions.

A compatible revision MUST NOT silently repurpose an existing function ID.

For example:

Function ID: 0x1001

Version 1:
calculate()

Version 2:
calculate()

is potentially compatible.

But:

Version 1:
0x1001 = calculate()

Version 2:
0x1001 = delete_account()

is incompatible.

An identifier MUST NOT be reused for a semantically different operation.


---

10. Function Signature Compatibility

A function's observable signature includes:

parameter count;

parameter order;

parameter identity;

parameter types;

parameter ownership;

parameter lifetime;

parameter mutability;

return type;

return ownership;

error model;

effects;

capabilities;

execution mode.


Changing any mandatory component MUST be evaluated for compatibility.

A change that can alter valid caller behavior MUST NOT be classified as backward compatible without a defined compatibility rule.


---

11. Parameter Compatibility

A newer implementation MUST continue accepting all parameter values that were valid under the older compatible contract, unless the old contract explicitly permitted rejection.

A parameter change is potentially breaking when it:

removes valid values;

changes representation;

changes interpretation;

changes ownership;

changes lifetime;

changes mutability;

introduces a new mandatory capability;

introduces a new mandatory effect.



---

12. Return-Value Compatibility

A newer implementation MUST preserve the semantic meaning of previously valid return values.

Adding information MAY be compatible when older consumers can safely ignore that information.

Changing the meaning of an existing return value is breaking.

For example:

Old:
0 = success

New:
0 = permission denied

is incompatible.


---

13. Error Compatibility

Error identities MUST remain stable across compatible revisions.

A newer implementation MAY introduce additional error values when the existing contract explicitly permits unknown errors.

However, it MUST NOT silently reinterpret an existing error ID.

Example:

Error 0x2001

must not mean:

Version 1:
PermissionDenied

Version 2:
Success

Error semantics are part of the compatibility contract.


---

14. Type Compatibility

A type is backward compatible when existing valid values retain their meaning and can still be safely interpreted.

Compatibility MUST consider:

type identity;

semantic meaning;

representation;

width;

signedness;

alignment;

encoding;

valid range;

ownership;

lifetime;

mutability.


Changing an internal representation MAY be compatible if the standardized boundary representation remains unchanged.


---

15. Primitive Type Compatibility

Primitive types MUST retain their established semantic meaning.

Examples:

Bool
Int
UInt
Float
Char
String
Bytes
Unit

A compatible revision MUST NOT silently change:

Int = signed integer

into:

Int = unsigned integer

without introducing a new incompatible contract.

Architecture-dependent sizes MUST NOT be used to silently alter meaning.


---

16. Numeric Compatibility

Numeric compatibility MUST preserve the valid range and interpretation of existing values.

A change MAY be backward compatible when it expands an explicitly extensible representation without changing the interpretation of existing values.

A change is potentially breaking when it:

narrows a range;

changes signedness;

changes overflow semantics;

changes byte order;

changes precision;

changes special-value semantics.



---

17. Record Compatibility

Records MUST identify fields using stable identifiers.

Compatible evolution MAY include:

adding optional fields;

adding fields with explicitly defined defaults;

retaining existing fields;

adding metadata that older implementations can safely ignore.


Compatible evolution MUST NOT:

silently change an existing field's meaning;

silently change an existing field's type;

remove a mandatory field;

reuse a field identifier for another meaning.


Example:

Version 1

Person {
    id
    name
}

may evolve to:

Version 2

Person {
    id
    name
    optional email
}

provided the contract explicitly defines the new field as optional and preserves the old interpretation.


---

18. Enum Compatibility

Enum identifiers MUST remain stable.

Adding a new enum value is compatible only when the original contract defines behavior for unknown values.

Otherwise the change is potentially breaking.

Existing enum values MUST NOT be reassigned.

Example:

Pending = 1
Active  = 2
Done    = 3

must not become:

Pending = 1
Deleted = 2
Done    = 3


---

19. Variant Compatibility

Variant identifiers MUST remain stable.

Adding a variant is backward compatible only when older implementations have a defined strategy for unknown variants.

Possible strategies include:

explicit unknown-variant handling;

rejection with a defined error;

extension preservation;

negotiated feature use.


A variant MUST NOT silently change its payload semantics.


---

20. Option Compatibility

Option<T> MUST preserve:

None
Some(value)

semantics.

A newer implementation MUST NOT reinterpret None as an ordinary value.

Existing Some(value) values MUST remain valid where the underlying type remains compatible.


---

21. Result Compatibility

Result<T,E> MUST preserve:

Success
Failure

semantics.

Existing success and error meanings MUST remain stable.

Additional error variants MAY be added when the contract permits them.


---

22. Handle Compatibility

Opaque handles MUST retain their defined semantic meaning.

A handle MUST NOT become valid for a different resource class without an explicitly incompatible interface revision.

For example:

Handle<File>

must not silently become:

Handle<NetworkSocket>


---

23. Memory Compatibility

Backward compatibility MUST preserve memory-boundary semantics.

These include:

ownership;

borrowing;

sharing;

mutability;

lifetime;

allocation;

release;

transfer.


A compatible revision MUST NOT silently transform:

Borrowed

into:

Transferred Ownership

or:

Immutable

into:

Mutable


---

24. Calling-Convention Compatibility

A newer implementation MUST preserve the applicable ULABI calling convention for existing consumers.

Changes to:

argument placement;

return placement;

stack requirements;

alignment;

register use;

ABI metadata;

calling mode;


are potentially breaking.

Platform-specific calling convention changes MUST be isolated behind the appropriate platform adapter.


---

25. Encoding Compatibility

Canonical serialized representations MUST remain readable by newer implementations when they claim backward compatibility.

A newer decoder MUST support all representations required by the older compatible contract.

Encoding changes SHOULD be additive where possible.

A canonical encoding MUST NOT silently change the meaning of an existing encoded value.


---

26. Decoding Safety

Backward compatibility MUST NOT require unsafe decoding.

A newer implementation MUST validate older data before:

allocating resources;

interpreting lengths;

dispatching variants;

accessing capabilities;

creating handles;

restoring state.


Malformed legacy data MUST produce a defined failure.


---

27. Capability Compatibility

Backward compatibility MUST include capability requirements.

A new implementation MUST NOT silently introduce a mandatory capability that older callers could not reasonably possess while claiming unconditional backward compatibility.

For example:

Version 1:
calculate()
Capabilities: None

Version 2:
calculate()
Capabilities: Network

is not unconditionally backward compatible.

It MAY be conditionally compatible if the new capability requirement is explicitly negotiated.


---

28. Effect Compatibility

Effects are part of the observable contract.

A newer implementation MUST NOT silently introduce effects that can change security, determinism, resource usage, or correctness.

Examples:

Pure

must not silently become:

Network

or:

WritesFilesystem

without an explicit contract change.


---

29. Locality Compatibility

Locality MUST remain explicit.

A function declared:

LocalOnly

MUST NOT silently become:

RemoteCapable

when that change affects:

latency;

failure;

authentication;

authorization;

consistency;

resource consumption.


A locality change is potentially breaking.


---

30. Execution-Semantics Compatibility

Changes to execution semantics MUST be evaluated.

Relevant properties include:

synchronous;

asynchronous;

blocking;

non-blocking;

cancellable;

streaming;

one-shot;

long-running;

idempotent;

non-idempotent.


A previously synchronous operation MUST NOT silently become asynchronous if that changes caller-visible behavior.


---

31. Security Compatibility

Security properties are part of compatibility.

A newer version MUST NOT silently weaken:

authentication;

authorization;

capability restrictions;

isolation;

integrity verification;

confidentiality guarantees;

validation requirements.


Security weakening MUST require an explicit incompatible or separately negotiated profile.


---

32. Profile Compatibility

Profiles MUST have independent compatibility identities.

A Core-compatible implementation is not automatically compatible with every profile.

For example:

ULABI Core 1.x

does not imply:

Streaming Profile
Security Profile
GPU Profile
Distributed Profile

Profile dependencies MUST be declared explicitly.


---

33. Additive Changes

The following changes MAY be backward compatible when all relevant conditions are satisfied:

adding optional fields;

adding optional metadata;

adding new error values under an extensible error model;

adding new enum values under an extensible enum model;

adding new variants under an extensible variant model;

adding optional capabilities;

adding optional profiles;

adding optional functions under an extensible interface;

increasing implementation capacity without changing old semantics.


Every additive change MUST preserve all previously valid behavior.


---

34. Potentially Breaking Changes

The following are potentially breaking:

removing a function;

removing a mandatory field;

changing a function ID;

changing a type ID;

changing type meaning;

changing signedness;

narrowing numeric ranges;

changing ownership;

changing lifetime;

changing mutability;

changing error semantics;

changing calling convention;

changing canonical encoding;

changing required capabilities;

adding mandatory effects;

changing execution semantics;

changing locality;

weakening security;

changing profile requirements.


Such changes MUST NOT be classified as backward compatible without an explicit specification rule proving compatibility.


---

35. Breaking Changes

A change is breaking when an older conforming consumer or producer can no longer safely interact with the newer implementation under the original contract.

Breaking changes SHOULD receive:

a new interface version;

a new major compatibility identity where applicable;

migration documentation;

compatibility tooling;

explicit release notes;

conformance tests.



---

36. Deprecation

Deprecation MUST be explicit.

A deprecated interface:

Existing
   |
   v
Deprecated
   |
   v
Removal Candidate

MUST remain valid for the compatibility period declared by its specification or profile.

Deprecation MUST NOT silently alter semantics.

A deprecated function SHOULD provide a machine-readable deprecation marker.


---

37. Removal

Removing a previously mandatory interface component is a breaking change unless an explicit compatibility mechanism preserves it.

Before removal, an implementation SHOULD provide:

replacement interface;

migration guidance;

deprecation period;

compatibility tests;

version transition rules.



---

38. Compatibility Shims

Implementations MAY provide compatibility shims.

A compatibility shim:

Legacy Interface
       |
       v
Compatibility Adapter
       |
       v
Current Interface

MUST preserve the semantics of the legacy interface.

A shim MUST NOT silently change:

ownership;

lifetime;

security;

error behavior;

capabilities;

execution semantics.


A shim MAY translate representations internally.


---

39. Adapter Requirements

Adapters between versions MUST explicitly document:

source version;

target version;

supported profiles;

translated types;

translated errors;

ownership transformations;

capability transformations;

unsupported operations;

loss of information;

security implications.


Lossy translation MUST be explicit.

An adapter MUST NOT claim full compatibility when it only provides partial translation.


---

40. Compatibility Direction

Compatibility MUST be directional where appropriate.

For example:

New Provider
      |
      v
Old Consumer

is a different compatibility question from:

Old Provider
      |
      v
New Consumer

Backward compatibility concerns the ability of newer implementations to preserve older contracts.

Forward compatibility is specified separately.


---

41. Consumer Preservation Rule

A newer provider claiming backward compatibility MUST preserve every previously valid interaction required by the older consumer contract.

Conceptually:

Old Consumer
      |
      | old contract
      v
New Provider
      |
      v
Compatible Behavior

The new provider MUST NOT require the old consumer to understand new mandatory semantics unless a negotiation mechanism explicitly establishes them.


---

42. Provider Preservation Rule

A newer consumer SHOULD continue to understand the behavior of older providers when the relevant interface is declared backward compatible.

However, this is distinct from provider-side backward compatibility and is subject to the forward-compatibility specification.


---

43. Unknown Extensions

Unknown optional extensions MUST NOT corrupt the interpretation of known data.

Where the contract allows extension preservation, unknown extensions SHOULD be preserved rather than discarded.

Where preservation is impossible, the loss MUST be explicit.


---

44. Reserved Identifiers

ULABI registries SHOULD reserve identifiers for future expansion.

Reserved identifiers MUST NOT be reused for incompatible meanings.

Identifier reuse can create catastrophic compatibility failures.


---

45. Compatibility Metadata

Machine-readable compatibility metadata SHOULD include:

interface_id
interface_version
core_version
profile_ids
profile_versions
type_schema_version
encoding_version
calling_convention_id
security_profile
capabilities
effects
locality
compatibility_level

This metadata SHOULD be available to validation and negotiation tooling.


---

46. Compatibility Verification

Compatibility MUST be verifiable.

Verification SHOULD include:

1. identifier comparison;


2. version comparison;


3. type comparison;


4. signature comparison;


5. ownership comparison;


6. lifetime comparison;


7. error comparison;


8. capability comparison;


9. effect comparison;


10. execution-semantics comparison;


11. encoding comparison;


12. security comparison;


13. profile dependency comparison.



A compatibility checker MUST NOT rely solely on source-code names.


---

47. ABI Difference Analysis

ULABI tooling SHOULD provide machine-readable ABI differences.

Example:

Interface: 0x1001

Version A:
calculate(Int) -> Result<Int, Error>

Version B:
calculate(Int) -> Result<Int, Error>

Status:
COMPATIBLE

Breaking example:

Version A:
calculate(Int) -> Result<Int, Error>

Version B:
calculate(UInt) -> Result<Int, Error>

Status:
INCOMPATIBLE

Reason:
Parameter signedness changed.


---

48. Compatibility Decision Algorithm

A conceptual compatibility checker SHALL follow:

Load Interface A
       |
Load Interface B
       |
Validate Metadata
       |
Compare Identity
       |
Compare Versions
       |
Compare Types
       |
Compare Functions
       |
Compare Errors
       |
Compare Ownership
       |
Compare Lifetimes
       |
Compare Effects
       |
Compare Capabilities
       |
Compare Execution Semantics
       |
Compare Encoding
       |
Compare Security
       |
Compare Profiles
       |
       v
Compatibility Result

Possible results:

COMPATIBLE
CONDITIONALLY_COMPATIBLE
INCOMPATIBLE
UNKNOWN


---

49. Compatibility Invariants

The following invariants MUST hold:

INV-BC-001

An existing interface identifier MUST NOT be reused for a different semantic contract.

INV-BC-002

Existing function identifiers MUST retain their semantic meaning.

INV-BC-003

Existing type identifiers MUST retain their semantic meaning.

INV-BC-004

Existing valid values MUST retain their meaning.

INV-BC-005

Existing ownership semantics MUST remain valid.

INV-BC-006

Existing lifetime semantics MUST remain valid.

INV-BC-007

Existing security guarantees MUST NOT be silently weakened.

INV-BC-008

Existing mandatory capabilities MUST NOT be silently expanded.

INV-BC-009

Existing canonical encodings MUST remain interpretable when backward compatibility is claimed.

INV-BC-010

Unknown compatibility MUST NOT be treated as guaranteed compatibility.

INV-BC-011

A compatibility shim MUST preserve observable semantics.

INV-BC-012

Internal implementation changes MUST NOT affect the standardized contract.


---

50. Failure Modes

Compatibility processing MUST define behavior for:

unknown interface;

unknown version;

malformed metadata;

unsupported type;

unsupported function;

incompatible type;

incompatible encoding;

incompatible calling convention;

incompatible capability;

incompatible security profile;

missing required profile;

ambiguous version;

invalid extension;

stale compatibility metadata.


Failures MUST be explicit.

Undefined compatibility behavior is prohibited.


---

51. Security Requirements

Compatibility mechanisms MUST NOT become a downgrade attack.

An implementation MUST prevent an attacker from forcing negotiation or compatibility processing toward an insecure legacy contract when a stronger contract is required by policy.

Examples of prohibited behavior include:

Supported:
Security Profile 3

Attacker requests:
Security Profile 1

Implementation silently downgrades.

If downgrade is permitted, it MUST be explicitly authorized by policy.


---

52. Legacy Security

Legacy interfaces MAY be retained for compatibility.

However, an implementation MUST be able to distinguish:

Legacy but permitted

from:

Legacy and prohibited

Security policy MAY reject older interfaces even when technical backward compatibility exists.

Therefore:

Technical Compatibility
        !=
Policy Authorization


---

53. Resource Compatibility

A newer implementation SHOULD preserve resource requirements of old contracts unless the specification explicitly allows changes.

A compatibility implementation MUST NOT silently increase resource usage beyond declared limits where doing so can violate correctness or safety.

Relevant resources include:

memory;

CPU;

storage;

handles;

file descriptors;

GPU resources;

network resources;

execution time.



---

54. Real-Time Compatibility

For real-time profiles, backward compatibility MUST include timing requirements.

Changes to:

worst-case execution time;

deadlines;

jitter;

blocking behavior;

scheduling requirements;


MAY be breaking even when type and function signatures remain unchanged.


---

55. Determinism Compatibility

A previously deterministic interface MUST NOT silently become nondeterministic when determinism is part of its contract.

Introducing:

randomness;

external time;

nondeterministic scheduling;

external state;


MUST be explicitly represented.


---

56. Distributed Compatibility

Distributed interfaces require additional compatibility checks for:

serialization;

transport;

retries;

idempotency;

timeout behavior;

consistency;

authentication;

authorization;

failure semantics.


A local compatibility guarantee MUST NOT automatically imply distributed compatibility.


---

57. Streaming Compatibility

For streaming interfaces, compatibility MUST include:

stream identity;

element type;

ordering;

completion semantics;

cancellation;

backpressure;

error behavior.


Changing any mandatory streaming semantic may be breaking.


---

58. Asynchronous Compatibility

For asynchronous interfaces, compatibility MUST include:

future/promise semantics;

completion;

cancellation;

error propagation;

scheduling guarantees;

resource lifetime.


A synchronous/asynchronous transition MUST NOT be assumed compatible.


---

59. Hardware Compatibility

Hardware-specific profiles MUST preserve the Core contract.

An accelerator implementation MAY provide optimized behavior.

It MUST NOT change the semantic meaning of a Core operation merely because the hardware implementation differs.

Architecture-specific optimizations belong in:

Platform Profiles
Hardware Profiles
Accelerator Profiles


---

60. Compatibility Across Languages

ULABI compatibility MUST remain independent of source language.

For example:

C
 |
ULABI
 |
Rust

and:

Python
 |
ULABI
 |
Java

must be evaluated using the same ULABI contract.

Source-language type systems MUST NOT determine universal compatibility.


---

61. Compatibility Across Runtimes

Different runtimes MAY implement the same ULABI interface.

For example:

Runtime A
   |
 ULABI
   |
Runtime B

Runtime differences MUST remain internal unless they affect observable ULABI semantics.


---

62. Compatibility Across Platforms

A compatible interface MAY be implemented on different:

operating systems;

CPU architectures;

devices;

virtual machines;

containers;

embedded environments.


Platform-specific restrictions MUST be expressed through profiles or capability declarations.


---

63. Compatibility Testing

Every compatibility-sensitive interface SHOULD have:

old-version fixtures;

new-version fixtures;

positive compatibility tests;

negative compatibility tests;

serialization tests;

ABI layout tests;

ownership tests;

lifetime tests;

error tests;

capability tests;

security tests.



---

64. Conformance Requirements

An implementation claiming backward compatibility MUST demonstrate:

1. preservation of interface identities;


2. preservation of function identities;


3. preservation of type semantics;


4. preservation of valid legacy values;


5. preservation of ownership;


6. preservation of lifetime;


7. preservation of errors;


8. preservation of required capabilities;


9. preservation of security requirements;


10. preservation of canonical representations where required.




---

65. Required Conformance Test Classes

The ULABI test suite SHOULD contain:

BC-IDENTITY
BC-FUNCTIONS
BC-TYPES
BC-RECORDS
BC-ENUMS
BC-VARIANTS
BC-ERRORS
BC-ENCODING
BC-CALLING-CONVENTION
BC-OWNERSHIP
BC-LIFETIME
BC-CAPABILITIES
BC-EFFECTS
BC-EXECUTION
BC-SECURITY
BC-PROFILES
BC-DEPRECATION
BC-SHIMS
BC-DOWNGRADE
BC-LEGACY


---

66. Reference Compatibility Matrix

A compatibility implementation SHOULD expose a matrix similar to:

Component	Old	New	Result

Interface ID	same	same	Compatible
Function IDs	same	same	Compatible
Existing types	same	same	Compatible
Optional field	absent	added	Compatible
Ownership	borrowed	borrowed	Compatible
Error IDs	preserved	preserved	Compatible
Capability	none	network required	Incompatible / Conditional
Calling convention	same	same	Compatible
Security	profile 2	profile 2	Compatible



---

67. Compatibility and Versioning

Backward compatibility MUST integrate with:

ULABI-VERSIONING.md

Version numbers MUST NOT be interpreted independently of interface and profile compatibility rules.

A version increment does not automatically mean incompatibility.

Likewise, retaining the same version number does not prove compatibility.

Compatibility MUST be established from the actual contract.


---

68. Compatibility and Feature Negotiation

Feature negotiation is specified by:

docs/compatibility/feature-negotiation.md

Backward compatibility establishes what older behavior means.

Negotiation establishes which mutually supported features may be used.

These concerns MUST remain separate.

Backward Compatibility
        |
        v
What old behavior remains valid?

Feature Negotiation
        |
        v
What optional behavior may both sides use?


---

69. Compatibility and Capability Discovery

Capability discovery is specified by:

docs/compatibility/capability-discovery.md

Capability discovery MAY provide information required to determine conditional compatibility.

Discovery MUST NOT itself grant authority.

Discover Capability
       |
       v
Evaluate Compatibility
       |
       v
Authorize Capability


---

70. Compatibility and Graceful Degradation

Graceful degradation is specified by:

docs/compatibility/graceful-degradation.md

A system MAY degrade functionality when a newer optional feature is absent.

However, degradation MUST NOT silently violate the old contract.


---

71. Compatibility and Forward Compatibility

Forward compatibility is specified separately in:

docs/compatibility/forwards-compatibility.md

The distinction is:

Backward Compatibility:
New implementation -> Old contract

Forward Compatibility:
Old implementation -> New contract

Both MAY exist simultaneously but MUST NOT be conflated.


---

72. No Reinterpretation Rule

A newer implementation MUST NOT reinterpret existing identifiers, types, errors, fields, or functions merely to preserve superficial compatibility.

If semantics must change:

Old Contract
     |
     +---- Preserve old contract
     |
     +---- Introduce new contract

The old identifier MUST NOT be repurposed.


---

73. Migration Strategy

A breaking revision SHOULD provide:

Old Interface
      |
      v
Compatibility Layer
      |
      v
New Interface

Migration documentation SHOULD identify:

old API;

new API;

mapping;

semantic differences;

capability differences;

ownership differences;

error differences;

unsupported cases;

testing procedure.



---

74. Compatibility Guarantees

A ULABI implementation MUST state the scope of its compatibility guarantee.

Example:

ULABI Core 1.2
Backward Compatibility:
Core 1.0 - 1.2

Profiles:
Security 2.x
Streaming 1.x

Unsupported:
GPU Profile 1.x

A generic statement such as:

ULABI Compatible

is insufficient for certification.


---

75. Independent Implementation Requirement

Backward compatibility MUST be implementable independently.

A conforming organization MUST NOT require access to:

another vendor's source code;

proprietary compiler internals;

proprietary runtime internals;

undocumented ABI behavior.


The published ULABI specification MUST provide sufficient information for compatibility implementation.


---

76. Reference Implementation Independence

A reference implementation MAY demonstrate compatibility.

However, the reference implementation MUST NOT become the normative definition of compatibility.

The specification remains authoritative.

Specification
     |
     +---- Implementation A
     |
     +---- Implementation B
     |
     +---- Implementation C

Not:

Implementation A
       |
       v
"Whatever A does is ULABI"


---

77. Compatibility Invariant Summary

A backward-compatible ULABI revision MUST satisfy:

Existing Identity
        +
Existing Meaning
        +
Existing Valid Inputs
        +
Existing Outputs
        +
Existing Errors
        +
Existing Ownership
        +
Existing Lifetime
        +
Existing Security
        +
Existing Required Capabilities
        =
Preserved Contract

If any mandatory semantic component changes incompatibly, the implementation MUST NOT claim unconditional backward compatibility.


---

78. Integration Contract

This document is intentionally self-contained.

It integrates with other ULABI specifications through stable conceptual contracts rather than requiring future rewriting.

Inputs consumed from other specifications

interface identity;

type identity;

version identity;

profile identity;

capability identity;

encoding identity;

calling-convention identity.


Outputs provided to other specifications

This document provides:

compatibility classification;

compatibility invariants;

compatibility rules;

breaking-change criteria;

legacy-preservation rules;

compatibility-test requirements.


Integration points

docs/abi/core-abi.md
        |
        v
Interface Contract
        |
        v
docs/compatibility/backwards-compatibility.md
        |
        +---- docs/compatibility/forwards-compatibility.md
        |
        +---- docs/compatibility/feature-negotiation.md
        |
        +---- docs/compatibility/capability-discovery.md
        |
        +---- docs/compatibility/graceful-degradation.md
        |
        +---- ULABI-VERSIONING.md
        |
        +---- docs/standards/conformance.md
        |
        +---- docs/standards/test-suite.md

No later specification is permitted to silently redefine the compatibility invariants established here.


---

79. Reference Implementation Requirements

A reference compatibility implementation SHOULD expose:

CompatibilityChecker
CompatibilityResult
InterfaceDiff
TypeDiff
FunctionDiff
ErrorDiff
EncodingDiff
OwnershipDiff
LifetimeDiff
CapabilityDiff
EffectDiff
SecurityDiff
ProfileDiff
MigrationReport

The implementation MAY be written in any programming language.


---

80. Final Rule

The fundamental ULABI backward-compatibility rule is:

> A newer implementation may change how an interface is implemented, but it MUST NOT silently change what an existing compatible ULABI contract means.



Therefore:

Implementation Changes
        |
        v
Allowed

Semantic Contract Changes
        |
        v
Compatibility Analysis Required

Breaking Semantic Changes
        |
        v
New Compatibility Identity / Explicit Migration

ULABI backward compatibility exists to preserve interoperability without freezing implementation technology.

The implementation may evolve.

The contract must remain trustworthy.

---

# 2. What this file owns — and what it deliberately does NOT own

This is important because you asked that we should not have to rewrite one document after another document is completed.

The ownership boundaries should now be:

| File | Owns |
|---|---|
| `ULABI-DESIGN.md` | Master architecture |
| `ULABI-SPEC.md` | Normative universal specification |
| `ULABI-VERSIONING.md` | Version numbering/version lifecycle |
| `docs/architecture/overview.md` | Architectural structure |
| `docs/architecture/design-principles.md` | Architectural principles |
| `docs/architecture/universal-model.md` | Universal abstraction model |
| `docs/architecture/scalability.md` | Scaling model |
| **`docs/compatibility/backwards-compatibility.md`** | **Preservation of existing contracts** |
| `docs/compatibility/forwards-compatibility.md` | Old implementations consuming newer contracts |
| `docs/compatibility/feature-negotiation.md` | Selecting mutually supported features |
| `docs/compatibility/capability-discovery.md` | Discovering capabilities |
| `docs/compatibility/graceful-degradation.md` | Safe reduced functionality |
| `docs/abi/core-abi.md` | Core ABI contract |
| `docs/abi/calling-convention.md` | ABI call mechanics |
| `docs/abi/data-types.md` | Boundary data representation |
| `docs/abi/memory-model.md` | Memory boundary semantics |
| `docs/standards/conformance.md` | What constitutes conformance |
| `docs/standards/test-suite.md` | Executable conformance tests |
| `docs/standards/certification.md` | Certification process |

This separation is particularly important because the repository already has the related compatibility files, even though several are currently placeholders. 

---

# 3. Required implementation/code architecture

The repository is currently primarily specification/documentation-oriented. The next implementation stage should **not** put implementation code into the documentation directories.

I recommend this language-neutral implementation architecture:

```text
implementations/
│
├── reference/
│   ├── core/
│   ├── types/
│   ├── calls/
│   ├── errors/
│   ├── encoding/
│   ├── compatibility/
│   └── validation/
│
├── runtime/
│   ├── interface/
│   ├── process/
│   ├── memory/
│   ├── resources/
│   └── capabilities/
│
├── adapters/
│   ├── language/
│   ├── platform/
│   ├── architecture/
│   ├── runtime/
│   └── transport/
│
└── profiles/
    ├── security/
    ├── async/
    ├── streaming/
    ├── distributed/
    ├── memory/
    ├── hardware/
    └── reliability/

Required code modules

Independent foundation modules — build first

These should have no dependency on Zamani, Sankofa, C, Rust, or another implementation language.

ulabi_core
ulabi_identity
ulabi_version
ulabi_types
ulabi_type_descriptor
ulabi_schema
ulabi_error
ulabi_result
ulabi_value
ulabi_handle
ulabi_encoding
ulabi_validation
ulabi_contract

ABI modules

ulabi_abi
ulabi_call
ulabi_calling_convention
ulabi_parameter
ulabi_return
ulabi_stack
ulabi_register
ulabi_exception

Memory modules

ulabi_memory
ulabi_ownership
ulabi_lifetime
ulabi_allocation
ulabi_buffer
ulabi_shared_memory
ulabi_zero_copy

Compatibility modules

These directly implement the specification you asked for:

ulabi_compatibility
ulabi_backward_compatibility
ulabi_forward_compatibility
ulabi_feature_negotiation
ulabi_capability_discovery
ulabi_graceful_degradation
ulabi_version_negotiation
ulabi_interface_diff
ulabi_schema_diff
ulabi_abi_diff
ulabi_migration
ulabi_compatibility_report

Security modules

ulabi_security
ulabi_capability
ulabi_authorization
ulabi_authentication
ulabi_isolation
ulabi_sandbox
ulabi_secure_loader
ulabi_integrity

Runtime modules

ulabi_runtime
ulabi_process
ulabi_thread
ulabi_async
ulabi_concurrency
ulabi_resource
ulabi_scheduler

Distributed modules

ulabi_transport
ulabi_serialization
ulabi_remote_call
ulabi_service_discovery
ulabi_distributed_error
ulabi_consensus_boundary

Reliability modules

ulabi_health
ulabi_fault_detection
ulabi_fault_isolation
ulabi_recovery
ulabi_rollback
ulabi_self_healing

Tooling

ulabi_validator
ulabi_abi_inspector
ulabi_schema_validator
ulabi_compatibility_checker
ulabi_contract_checker
ulabi_conformance_runner
ulabi_diagnostics
ulabi_trace
ulabi_profiler


---

4. Schemas that should eventually exist

The compatibility specification requires machine-readable metadata, so these should eventually be created under:

schemas/

Recommended:

schemas/interface.schema.json
schemas/version.schema.json
schemas/profile.schema.json
schemas/type.schema.json
schemas/function.schema.json
schemas/error.schema.json
schemas/capability.schema.json
schemas/effect.schema.json
schemas/encoding.schema.json
schemas/calling-convention.schema.json
schemas/compatibility.schema.json
schemas/compatibility-result.schema.json
schemas/abi-diff.schema.json
schemas/migration.schema.json
schemas/conformance.schema.json

The important point is that the Markdown specification is normative; schemas encode the specification for machines.


---

5. Required test files

The compatibility subsystem should eventually have:

tests/
├── compatibility/
│   ├── backward_identity_tests
│   ├── backward_function_tests
│   ├── backward_type_tests
│   ├── backward_record_tests
│   ├── backward_enum_tests
│   ├── backward_variant_tests
│   ├── backward_error_tests
│   ├── backward_encoding_tests
│   ├── backward_calling_convention_tests
│   ├── backward_ownership_tests
│   ├── backward_lifetime_tests
│   ├── backward_capability_tests
│   ├── backward_effect_tests
│   ├── backward_execution_tests
│   ├── backward_security_tests
│   ├── backward_profile_tests
│   ├── backward_deprecation_tests
│   ├── backward_shim_tests
│   └── backward_downgrade_tests

And the conformance layer:

conformance/
├── core/
├── types/
├── abi/
├── memory/
├── compatibility/
├── security/
├── runtime/
├── distributed/
├── reliability/
└── profiles/


---

6. Integration order — so files don't need circular rewriting

This is the order I recommend for the entire ULABI implementation/specification program.

Phase 1 — completely independent foundations

1. ULABI-DESIGN.md


2. docs/architecture/design-principles.md


3. docs/architecture/overview.md


4. docs/architecture/universal-model.md


5. docs/architecture/scalability.md


6. ULABI-VERSIONING.md


7. schemas/version.schema.json


8. schemas/interface.schema.json


9. schemas/profile.schema.json



These define the vocabulary and architectural boundaries.

Phase 2 — Core ABI

10. docs/abi/core-abi.md


11. docs/abi/data-types.md


12. docs/abi/calling-convention.md


13. docs/abi/return-values.md


14. docs/abi/exception-model.md


15. docs/abi/stack-model.md


16. docs/abi/register-model.md


17. docs/abi/memory-model.md



Phase 3 — compatibility

18. docs/compatibility/backwards-compatibility.md


19. docs/compatibility/forwards-compatibility.md


20. docs/compatibility/feature-negotiation.md


21. docs/compatibility/capability-discovery.md


22. docs/compatibility/graceful-degradation.md



The document I wrote above is designed so it can be completed now, without waiting for those five documents.

Phase 4 — interoperability

23. docs/interoperability/language-interoperability.md


24. docs/interoperability/foreign-function-interface.md


25. docs/interoperability/object-model.md


26. docs/interoperability/name-mangling.md


27. docs/interoperability/symbol-resolution.md


28. docs/interoperability/cross-language-data.md



Phase 5 — runtime and memory

29. runtime specifications


30. memory specifications


31. resource specifications


32. async/concurrency specifications



Phase 6 — security

33. security model


34. capability security


35. sandboxing


36. authentication


37. authorization


38. secure loading


39. supply-chain security



Phase 7 — distributed and hardware

40. distributed ABI


41. serialization


42. remote calls


43. service discovery


44. CPU


45. GPU


46. NPU


47. FPGA


48. quantum/future architectures



Phase 8 — conformance

49. docs/standards/conformance.md


50. docs/standards/compliance-levels.md


51. docs/standards/test-suite.md


52. docs/standards/certification.md


53. docs/standards/reference-implementations.md




---

7. Most important architectural consequence

The current repository already has the right high-level separation: ULABI-DESIGN.md explicitly says ULABI is an interoperability contract rather than a programming language, and specifically prevents Zamani or Sankofa from becoming part of ULABI itself.

The compatibility system should therefore look like:

ULABI
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      Core        Profiles      Compatibility
        │             │             │
        │             │       ┌─────┼─────┐
        │             │       │     │     │
        │             │      Back  Forward Negotiation
        │             │
        └─────────────┴──────────────┐
                                     │
                              Implementations
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
                  C                Rust             Python
                  │                  │                  │
                Adapter            Adapter           Adapter
                  │                  │                  │
                  └──────────────── ULABI ──────────────┘

That is the critical boundary: ULABI owns the contract; implementations own the mechanisms.

Current repository status relevant to this task

ULABI-DESIGN.md: substantial master architecture, 0.2.0-draft.

docs/architecture/overview.md: already substantially drafted rather than a blank file.

docs/architecture/design-principles.md: already drafted in the repository history/current content rather than needing a conceptual redesign.

docs/compatibility/backwards-compatibility.md: currently a placeholder and is the correct target for the specification above.

The remaining compatibility documents exist separately, which allows us to keep each document single-purpose.


Do not create the implementation modules yet just because their names are listed. The next step should be to finish the normative contracts and schemas first, then implement the code directly against those frozen contracts. This prevents exactly the re-editing/integration problem you are trying to eliminate.