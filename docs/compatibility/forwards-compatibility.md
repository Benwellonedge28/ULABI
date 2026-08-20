I read the repository’s current ULABI-DESIGN.md first and inspected the existing architecture/ABI/compatibility documents and repository tree before defining this file. The master design establishes Minimal Core + Standard Profiles + Extensible Ecosystem, stable contracts, language/runtime/platform neutrality, explicit semantics, versioning, and implementation independence.

I also found that docs/compatibility/backwards-compatibility.md, capability-discovery.md, and feature-negotiation.md already exist, while forwards-compatibility.md is currently only a placeholder.   The repository already has substantial ABI specifications such as core ABI, calling convention, data types, memory, stack, registers, and return values, so the forward-compatibility document should not redefine those contracts.

docs/compatibility/forwards-compatibility.md

Copy everything below into that file.

# ULABI Forward Compatibility

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Document:** `docs/compatibility/forwards-compatibility.md`  
**Specification Family:** Compatibility  
**Status:** Normative Specification  
**Specification Version:** 0.1.0-draft  
**ULABI Architecture Version:** 0.2.0-draft  
**Normative Authority:** `ULABI-DESIGN.md`, `ULABI-SPEC.md`, `ULABI-VERSIONING.md`  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the requirements for forward compatibility in ULABI.

Forward compatibility means that an implementation, consumer, decoder,
validator, runtime, or tool can safely interact with a newer ULABI contract,
message, interface, type evolution, profile, or extension without requiring
the older implementation to understand every newly introduced feature.

Forward compatibility is distinct from backward compatibility.

Backward compatibility asks:

> Can a newer implementation continue serving an older contract?

Forward compatibility asks:

> Can an older implementation safely encounter a newer contract or data
> representation without misinterpreting it, corrupting state, violating
> security requirements, or silently changing semantics?

ULABI forward compatibility MUST be explicit.

An implementation MUST NOT claim forward compatibility merely because it can
ignore unknown bytes or fields.

Safe forward compatibility requires defined handling of unknown information.

---

# 2. Scope

This specification governs:

- interface evolution;
- schema evolution;
- type evolution;
- record evolution;
- enum evolution;
- variant evolution;
- error evolution;
- optional extensions;
- unknown fields;
- unknown operations;
- unknown capabilities;
- unknown profiles;
- unknown metadata;
- version identifiers;
- extension preservation;
- decoding behavior;
- validation behavior;
- feature discovery;
- negotiated behavior;
- graceful rejection;
- safe degradation.

This specification does NOT redefine:

- the core calling convention;
- the memory model;
- primitive type semantics;
- complete binary encoding rules;
- capability discovery mechanisms;
- feature negotiation mechanisms;
- graceful degradation policy;
- certification procedures.

Those responsibilities belong to their dedicated specifications.

---

# 3. Architectural Authority

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.

Forward compatibility is therefore achieved primarily through:

1. stable identifiers;
2. additive evolution;
3. explicit extension points;
4. optional capabilities;
5. unknown-value handling;
6. deterministic rejection;
7. version-aware validation;
8. preservation of information where required;
9. explicit negotiation;
10. security-first failure behavior.

Forward compatibility MUST NOT be achieved by silently changing the meaning
of existing ULABI constructs.

The compatibility architecture is:

```text
                    ULABI-DESIGN.md
                           |
                    ULABI-SPEC.md
                           |
                  ULABI-VERSIONING.md
                           |
             +-------------+-------------+
             |                           |
      Backward Compatibility      Forward Compatibility
             |                           |
             +-------------+-------------+
                           |
                Feature Negotiation
                           |
                Capability Discovery
                           |
                Graceful Degradation


---

4. Normative Language

The following terms are normative:

MUST

MUST NOT

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY


A conforming implementation MUST satisfy every applicable MUST and MUST NOT requirement.


---

5. Fundamental Principle

Forward compatibility MUST preserve safety and semantic integrity.

An older implementation encountering newer information MUST never be forced to guess its meaning.

Therefore:

Unknown
   |
   +-- safely ignorable
   |
   +-- safely preservable
   |
   +-- safely rejectable
   |
   +-- requires negotiation
   |
   +-- security-critical -> reject

Unknown information MUST NOT automatically be treated as valid.

Unknown information MUST NOT automatically be treated as invalid.

Its handling depends on the contract governing that information.


---

6. Forward Compatibility Classes

ULABI defines four outcomes:

1. Forward Compatible


2. Conditionally Forward Compatible


3. Safely Rejectable


4. Forward Incompatible



6.1 Forward Compatible

An older implementation can safely process the newer contract without modification and without loss of required semantics.


---

6.2 Conditionally Forward Compatible

The older implementation can safely operate only when specified conditions are satisfied.

Examples:

an extension is optional;

the sender avoids unsupported features;

unknown fields are safely ignorable;

a common profile is selected;

the newer implementation provides a legacy representation.


Conditions MUST be explicit.


---

6.3 Safely Rejectable

The older implementation cannot process the newer feature but can detect and reject it deterministically without corrupting state or violating security.

Safe rejection is a valid forward-compatibility behavior.

A system MUST NOT be considered forward-compatible merely because it crashes or rejects malformed input.

The rejection MUST be:

deterministic;

bounded;

observable;

non-destructive;

security-safe.



---

6.4 Forward Incompatible

The newer contract cannot be safely processed or rejected by the older implementation.

Examples include:

unknown information changes the interpretation of known data;

the newer representation violates the older decoder's safety assumptions;

an unknown operation can be executed accidentally;

an extension changes an existing field's semantics;

an unknown security requirement cannot be enforced.



---

7. Stable Identifiers

Forward-compatible evolution MUST use stable identifiers.

Identifiers include:

interface IDs;

function IDs;

type IDs;

field IDs;

enum IDs;

variant IDs;

error IDs;

capability IDs;

profile IDs;

extension IDs.


Identifiers MUST NOT be inferred solely from:

source names;

declaration order;

memory addresses;

compiler-generated names;

implementation-specific symbol names.


An unknown identifier MUST be distinguishable from a known identifier.


---

8. Additive Evolution

ULABI SHOULD prefer additive evolution.

Examples of potentially forward-compatible additions include:

optional fields;

optional metadata;

new capabilities;

new profiles;

new enum values when unknown values are defined;

new variants when unknown variants are defined;

new extension blocks;

new diagnostics;

new non-required attributes.


Existing meanings MUST remain unchanged.

Additive evolution MUST NOT be used to disguise a semantic breaking change.


---

9. Existing Semantics Are Immutable

A newer specification MUST NOT redefine an existing identifier.

For example:

Version 1:

0x1001 = calculate

must not become:

Version 2:

0x1001 = delete

Similarly:

Field 7 = account_id

must never become:

Field 7 = encryption_key

An identifier's semantic identity is part of the ULABI contract.


---

10. Unknown Fields

Records SHOULD support explicitly defined unknown-field behavior.

An older implementation MAY ignore an unknown field only when the contract declares that doing so is safe.

For example:

Version 1

Person {
    id
    name
}

may encounter:

Version 2

Person {
    id
    name
    email
}

if email is explicitly optional and unknown fields are declared ignorable.

An unknown field MUST NOT be ignored when:

it changes interpretation of known fields;

it carries mandatory security information;

it controls authorization;

it changes ownership;

it changes memory lifetime;

it changes execution semantics;

it changes integrity requirements.



---

11. Unknown Field Preservation

Some protocols require intermediaries to preserve information they do not understand.

Where preservation is required, the implementation MUST retain unknown fields without changing their:

identifier;

encoding;

ordering semantics where ordering is significant;

integrity metadata;

extension metadata.


An implementation MUST NOT claim transparent forwarding if it silently drops required unknown information.


---

12. Unknown Enum Values

An older implementation may encounter an enum value introduced by a newer version.

Example:

Version 1:

Pending = 1
Active  = 2
Done    = 3

Version 2:

Pending = 1
Active  = 2
Done    = 3
Paused  = 4

Forward compatibility exists only if the original contract defines safe handling of unknown values.

Possible behaviors include:

preserve;

map to an explicit Unknown state;

reject;

negotiate an older value.


An unknown enum value MUST NOT silently map to a semantically unrelated known value.


---

13. Unknown Variants

Variants MUST define their unknown-variant behavior when forward evolution is expected.

Possible behavior:

Known Variant
       |
       +-- process

Unknown Variant
       |
       +-- preserve
       |
       +-- reject
       |
       +-- negotiated fallback

An unknown variant MUST NOT be interpreted as an existing variant merely because its binary representation happens to resemble one.


---

14. Unknown Operations

Unknown function or operation identifiers MUST be rejected unless the contract explicitly defines an extension mechanism.

An unknown operation MUST NOT:

execute by default;

map to a nearby operation;

fall back to an unrelated operation;

trigger privileged behavior.


Default behavior for an unknown operation SHOULD be a deterministic "unsupported operation" result.


---

15. Unknown Errors

An older implementation may receive an error introduced by a newer version.

Unknown errors MUST remain distinguishable from known errors.

An implementation MAY expose:

UnknownError {
    identifier
    optional payload
}

where supported by the error model.

Unknown errors MUST NOT silently become:

Success

or another unrelated error.


---

16. Type Evolution

Forward-compatible type evolution MUST preserve the semantic identity of existing types.

A newer type system MAY add:

optional metadata;

optional fields;

extensions;

additional variants;

additional annotations.


It MUST NOT silently change:

signedness;

numeric interpretation;

ownership;

lifetime;

mutability;

encoding;

security meaning.


A newer representation MUST remain distinguishable from an older representation.


---

17. Numeric Evolution

Numeric types require special care.

A newer implementation MUST NOT assume that an older implementation can handle a wider range merely because the underlying binary representation can store it.

For example:

UInt32

cannot become:

UInt128

without defining what happens when values exceed the older implementation's range.

Possible safe behavior:

reject;

constrain the transmitted range;

negotiate a compatible representation;

use an explicitly extensible numeric type.


Overflow MUST never become silent truncation unless explicitly defined by the contract.


---

18. String and Byte Evolution

Strings and byte sequences MUST remain distinguishable.

A newer implementation MUST NOT assume that an older implementation supports:

a new encoding;

a new normalization requirement;

arbitrary size increases;

new control semantics embedded in strings.


Unknown string metadata MUST NOT change the interpretation of existing bytes without explicit versioning.


---

19. Record Evolution

Records SHOULD be designed for additive evolution.

Forward-compatible records should use:

stable field identifiers;

explicit field types;

optionality metadata;

default semantics where appropriate;

unknown-field behavior;

maximum-size constraints.


A newer producer MUST NOT require an older consumer to understand a newly added mandatory field unless negotiation has established support.


---

20. Interface Evolution

A new interface revision SHOULD preserve the original interface identity when its contract remains compatible.

A breaking semantic change SHOULD receive a new interface identity or explicit incompatible version.

An implementation MUST NOT hide incompatible evolution behind an unchanged identifier.


---

21. Extension Mechanism

ULABI extensions MUST be explicitly identifiable.

An extension SHOULD contain:

Extension ID
Extension Version
Length
Flags
Payload

where applicable.

Unknown extensions MUST have explicitly defined behavior.

An extension MUST NOT modify Core semantics unless the Core specification explicitly permits that extension point.


---

22. Extension Isolation

Extensions MUST remain isolated from unrelated Core contracts.

For example:

ULABI Core
    |
    +-- Security Extension
    |
    +-- Streaming Extension
    |
    +-- GPU Extension
    |
    +-- Distributed Extension

The presence of one extension MUST NOT silently imply the presence of another.

Dependencies MUST be declared.


---

23. Capability-Based Forward Compatibility

Capabilities provide a mechanism for discovering whether newer functionality can safely be used.

A capability MUST have:

stable identifier;

semantic definition;

version;

security classification;

dependency information where applicable.


An older implementation MUST NOT assume an unknown capability is supported.

Capability discovery is specified by:

docs/compatibility/capability-discovery.md

Feature selection is specified by:

docs/compatibility/feature-negotiation.md

This document defines their forward-compatibility consequences, not their complete protocols.


---

24. Version Handling

ULABI versions MUST be explicit.

An implementation MUST be able to distinguish:

Known Version
Unknown Version
Unsupported Version
Compatible Version
Incompatible Version

Version comparison MUST NOT rely solely on lexical string comparison.

Version semantics are governed by:

ULABI-VERSIONING.md


---

25. Major Version Changes

A major version MAY contain breaking changes.

Older implementations MUST NOT assume compatibility across a major version boundary.

Where interoperability is still possible, an explicit compatibility profile, adapter, or negotiated legacy mode SHOULD be used.


---

26. Minor Version Changes

Minor-version additions SHOULD be designed to preserve existing semantics.

They MAY add:

optional features;

optional fields;

extensions;

new diagnostics;

new capabilities.


They MUST NOT silently redefine existing mandatory behavior.


---

27. Patch Changes

Patch-level changes SHOULD preserve the complete observable contract.

Patch releases SHOULD be limited to:

corrections;

clarifications;

implementation fixes;

non-semantic documentation changes.


A patch release MUST NOT introduce an undocumented breaking semantic change.


---

28. Unknown Metadata

Unknown metadata MAY be ignored only when its contract explicitly declares that it is non-semantic.

Metadata MUST NOT be considered harmless merely because it is labeled "metadata."

Security, authorization, ownership, integrity, and execution metadata may be semantically significant.


---

29. Security-Critical Unknown Information

Security-critical unknown information MUST fail closed.

Examples:

unknown authorization requirement;

unknown cryptographic mode;

unknown integrity requirement;

unknown capability restriction;

unknown isolation requirement;

unknown trust level.


An older implementation MUST NOT guess a permissive interpretation.

Therefore:

Unknown Security Requirement
          |
          v
       Reject

is the default behavior unless a stronger safe rule is explicitly defined.


---

30. Integrity and Authentication

Forward compatibility MUST NOT weaken integrity or authentication.

An older implementation MUST NOT accept a newer representation merely because it cannot validate newly introduced security metadata.

If validation is mandatory and unsupported:

Unsupported Security Requirement
              |
              v
            Reject


---

31. Resource Limits

Forward-compatible decoding MUST remain bounded.

A newer producer MUST NOT be able to exploit unknown fields, extensions, or future versions to cause unbounded:

memory allocation;

recursion;

CPU consumption;

nesting;

buffering;

concurrency;

handle creation.


Implementations MUST apply resource limits before processing untrusted future-version data.


---

32. Safe Decoding Algorithm

A conforming decoder SHOULD follow this conceptual sequence:

Receive
   |
Validate envelope
   |
Validate version
   |
Validate size/resource limits
   |
Validate known identifiers
   |
Classify unknown information
   |
+--+----------------------+
|                         |
Safe known            Unknown
|                         |
Process              +----+----+
                     |         |
                  Ignore    Preserve
                     |         |
                     +----+----+
                          |
                     Or Reject
                          |
                       Verify
                          |
                       Execute

No execution MUST occur before required validation has completed.


---

33. Future Data Must Not Become a Security Boundary Bypass

Forward compatibility MUST NOT allow newer data to bypass:

authorization;

capability checks;

sandboxing;

memory validation;

resource limits;

integrity checks;

authentication.


Unknown fields MUST NOT become an implicit privilege channel.


---

34. Compatibility Through Adapters

An older implementation MAY use an adapter to communicate with a newer implementation.

The adapter MUST explicitly define:

supported versions;

translated fields;

dropped fields;

preserved fields;

unsupported features;

security implications;

semantic limitations.


An adapter MUST NOT silently claim complete compatibility when it performs lossy translation.


---

35. Lossy Translation

Lossy translation MUST be detectable.

For example:

New Contract
     |
     | field X unsupported
     v
Older Contract

The adapter MUST either:

preserve X;

explicitly discard X under an allowed rule;

report the loss;

reject the operation.


Silent semantic loss is prohibited.


---

36. Forward Compatibility and Determinism

When unknown information is safely ignored, the resulting behavior SHOULD be deterministic.

Two conforming implementations receiving the same compatible future input should produce equivalent observable behavior.

Unknown information MUST NOT introduce hidden nondeterminism.


---

37. Streaming and Forward Compatibility

For streaming interfaces, implementations MUST be able to determine whether future records can be:

safely ignored;

preserved;

rejected;

negotiated.


An unknown stream item MUST NOT corrupt stream framing.

A future extension MUST NOT make it impossible for an older implementation to identify the boundaries of subsequent items.


---

38. Distributed Forward Compatibility

Distributed implementations MUST treat forward compatibility as a protocol property, not merely a local decoding property.

A node MUST NOT assume that every peer understands:

every interface;

every profile;

every capability;

every extension;

every version.


Unsupported features MUST produce defined protocol behavior.

Forward compatibility MUST remain independent of the transport.


---

39. State and Persistent Data

Persistent ULABI data SHOULD use versioned schemas.

Older readers encountering newer persistent state MUST:

validate the version;

apply defined migration rules;

preserve unknown data where required;

reject unsafe states.


A migration MUST NOT silently reinterpret persisted data.


---

40. State Migration

Migration procedures MUST define:

Source Version
Target Version
Migration Preconditions
Transformation
Validation
Failure Behavior
Rollback / Recovery

A migration MUST be idempotent where the contract requires retryability.

Partial migration MUST NOT leave a system in an ambiguous state.


---

41. Forward-Compatible Configuration

Configuration fields SHOULD have explicit unknown-field rules.

Unknown configuration MUST NOT automatically enable functionality.

Security-sensitive unknown configuration MUST be rejected unless explicitly defined as ignorable.


---

42. Forward Compatibility and Optionality

Optionality MUST be explicit.

These are different:

Optional

and:

Unknown

An unknown field is not automatically optional.

An optional field is one whose absence and unsupported status have defined semantics.


---

43. Default Values

Defaults MUST be explicit.

A newer producer MUST NOT assume an older implementation knows a new default.

If an older implementation cannot determine a safe default, it MUST reject or negotiate.

Defaults MUST NOT silently weaken security.


---

44. Graceful Failure

Forward-compatible failure MUST be predictable.

Recommended behavior:

Future Feature
      |
Supported?
  +---+---+
 YES     NO
 |        |
Use    Safe fallback?
          +---+---+
         YES     NO
          |       |
       Fallback  Reject

Graceful degradation is specified separately in:

docs/compatibility/graceful-degradation.md


---

45. Forward Compatibility Is Not Silent Downgrading

An implementation MUST NOT silently downgrade to an older feature when doing so changes:

security;

correctness;

durability;

authorization;

data integrity;

semantics.


Downgrades MUST be explicit or governed by a defined policy.


---

46. Negotiation Boundary

Forward compatibility and feature negotiation are related but distinct.

This specification establishes:

> Unknown future functionality must be safely classified.



feature-negotiation.md establishes:

> How implementations determine which functionality both parties can use.



Neither document should duplicate the other's protocol definitions.


---

47. Capability Discovery Boundary

Capability discovery establishes what an implementation can provide or accept.

Forward compatibility establishes how unknown future capabilities are handled.

An implementation MUST NOT infer support from:

implementation version alone;

source-language identity;

operating system;

CPU architecture;

vendor;

compiler;

runtime.


Support MUST be established through the defined compatibility mechanisms.


---

48. Compatibility Matrix

Implementations SHOULD maintain a machine-readable compatibility matrix.

Example:

Feature	Known	Supported	Required	Safe to Ignore

Core Types	Yes	Yes	Yes	No
New Field	No	No	No	Yes
New Capability	No	No	No	No
New Security Requirement	No	No	Yes	No
New Optional Extension	No	No	No	Yes


The matrix MAY be generated from interface metadata.


---

49. Compatibility Evidence

A compatibility claim SHOULD identify its evidence.

Evidence MAY include:

interface schema;

version metadata;

capability declarations;

feature-negotiation result;

conformance tests;

compatibility test results;

formal verification;

validator output.


"Works in practice" alone is insufficient for a normative compatibility claim.


---

50. Compatibility Validator

ULABI tooling SHOULD be capable of comparing:

Old Contract
      |
      v
Compatibility Analyzer
      ^
      |
New Contract
      |
      v
Compatibility Result

Possible results:

FORWARD_COMPATIBLE
CONDITIONALLY_COMPATIBLE
SAFELY_REJECTABLE
INCOMPATIBLE
UNKNOWN

The analyzer MUST distinguish semantic changes from internal implementation changes.


---

51. Required Conformance Tests

A forward-compatible implementation MUST be tested against:

1. unknown fields;


2. unknown enum values;


3. unknown variants;


4. unknown extensions;


5. unknown capabilities;


6. unknown operations;


7. unknown errors;


8. newer minor versions;


9. unsupported major versions;


10. malformed future data;


11. oversized future data;


12. security-critical unknown metadata;


13. preserved extensions;


14. safely ignorable extensions;


15. negotiated future features;


16. rejected future features;


17. adapter-based translation;


18. lossy translation;


19. persistent-state migration;


20. downgrade attempts.




---

52. Negative Tests

Conformance testing MUST include negative cases.

Examples:

Unknown operation -> reject
Unknown security requirement -> reject
Unknown capability -> no implicit privilege
Unknown mandatory field -> reject
Invalid future encoding -> reject
Oversized extension -> reject
Invalid version -> reject
Semantic identifier reuse -> incompatible


---

53. Security Test Requirements

Tests MUST verify that future-version data cannot:

bypass authorization;

bypass capability checks;

cause arbitrary code execution;

cause uncontrolled allocation;

cause unbounded recursion;

cause resource exhaustion;

alter ownership;

alter memory lifetime;

disable integrity verification;

disable authentication.



---

54. Reference State Machine

The normative conceptual state machine is:

Receive Future Contract
                       |
                       v
                 Validate Envelope
                       |
                       v
                 Validate Version
                       |
                       v
                Validate Limits
                       |
                       v
               Identify Features
                       |
             +---------+---------+
             |                   |
           Known              Unknown
             |                   |
          Supported?        Safe handling?
          +---+---+          +---+---+
         Yes     No         Yes     No
          |       |          |       |
        Use    Negotiate   Ignore/  Reject
                            Preserve
          |       |          |
          +---+---+----------+
              |
           Validate
              |
           Execute
              |
           Verify
              |
             Done

Security-critical unknown requirements follow the rejection path.


---

55. Prohibited Behaviors

A conforming implementation MUST NOT:

guess the meaning of unknown identifiers;

reinterpret unknown values as known values;

execute unknown operations;

silently weaken security;

silently drop mandatory future information;

silently truncate future numeric values;

silently downgrade security;

silently change ownership;

silently change memory lifetime;

treat unknown as supported;

claim compatibility without evidence.



---

56. Relationship to Backward Compatibility

Backward compatibility answers:

New implementation
        |
        v
Old contract

Forward compatibility answers:

Old implementation
        |
        v
New contract

The two properties MUST be evaluated independently.

A system MAY be:

backward compatible but not forward compatible;

forward compatible but not backward compatible;

both;

neither.


No implementation MUST infer one property from the other.


---

57. Relationship to Versioning

ULABI-VERSIONING.md defines version identifiers and version lifecycle.

This document defines how version differences affect safe future evolution.

Version numbers alone MUST NOT establish compatibility.

Compatibility depends on the observable contract.


---

58. Relationship to Conformance

docs/standards/conformance.md defines overall conformance.

A forward-compatibility claim MUST identify:

supported ULABI versions;

supported profiles;

supported extensions;

unknown-feature behavior;

migration behavior;

rejection behavior.



---

59. Reference Implementation Requirements

A future ULABI reference implementation SHOULD provide:

version-aware decoder;

unknown-field handling;

extension registry;

compatibility classifier;

capability integration;

negotiation integration;

bounded decoding;

compatibility diagnostics;

conformance test fixtures.


The reference implementation MUST NOT become the normative authority.

The specification remains authoritative.


---

60. Machine-Readable Contract

Where practical, forward-compatibility rules SHOULD be represented in schemas.

A compatibility descriptor MAY contain:

interface_id
interface_version
minimum_supported_version
maximum_supported_version
supported_profiles
supported_extensions
unknown_field_policy
unknown_variant_policy
unknown_error_policy
unknown_operation_policy
security_requirements
migration_policy
degradation_policy

The exact schema belongs in the ULABI schema system.


---

61. Integration Contract

This document integrates with the repository as follows:

ULABI-DESIGN.md
       |
       +---- ULABI-SPEC.md
       |
       +---- ULABI-VERSIONING.md
       |
       +---- backwards-compatibility.md
       |
       +---- forwards-compatibility.md
       |
       +---- feature-negotiation.md
       |
       +---- capability-discovery.md
       |
       +---- graceful-degradation.md
       |
       +---- conformance.md
       |
       +---- test-suite.md

Ownership boundaries:

Concern	Authoritative File

Overall architecture	ULABI-DESIGN.md
Normative Core	ULABI-SPEC.md
Version numbering	ULABI-VERSIONING.md
Older contract preservation	backwards-compatibility.md
Newer contract handling	forwards-compatibility.md
Feature selection	feature-negotiation.md
Capability enumeration	capability-discovery.md
Fallback behavior	graceful-degradation.md
Conformance	conformance.md
Test methodology	test-suite.md


No later document needs to modify this document merely because another compatibility document is implemented.


---

62. Implementation Independence

ULABI forward compatibility MUST remain independent of:

programming language;

compiler;

linker;

runtime;

operating system;

CPU;

GPU;

NPU;

FPGA;

transport;

vendor;

organization.


Zamani, Sankofa, C, C++, Rust, Go, Python, Java, Swift, Kotlin, Fortran, Ada, and other systems MAY independently implement these rules.

ULABI does not define how they implement them internally.


---

63. Final Requirements

A conforming ULABI implementation claiming forward compatibility MUST:

1. identify supported versions;


2. validate version information;


3. distinguish known from unknown identifiers;


4. define unknown-field behavior;


5. define unknown-extension behavior;


6. define unknown-operation behavior;


7. define unknown-error behavior;


8. define unknown-variant behavior;


9. enforce resource limits;


10. fail closed on unsupported security requirements;


11. never guess unknown semantics;


12. preserve information where required;


13. expose unsupported features deterministically;


14. integrate with capability discovery;


15. integrate with feature negotiation;


16. integrate with graceful degradation;


17. pass applicable forward-compatibility conformance tests.



The fundamental rule is:

> Future functionality may be unknown, but it must never be allowed to become ambiguous, unsafe, or silently destructive.




---

64. Conformance Statement

An implementation MAY claim:

ULABI Forward Compatibility: PASS

only when it satisfies the applicable requirements of this specification and passes the corresponding conformance tests.

A stronger claim SHOULD identify the supported compatibility scope:

ULABI Core
Forward Compatible: Yes

Unknown Fields: Preserve
Unknown Extensions: Ignore
Unknown Operations: Reject
Unknown Security Requirements: Reject
Future Minor Versions: Supported
Future Major Versions: Safely Rejectable

Compatibility claims MUST be precise rather than simply stating:

ULABI Compatible

## What this file deliberately does **not** duplicate

The repository already has separate specifications for the ABI core, calling convention, data types, memory model, stack/register models, and return values.    

So `forwards-compatibility.md` owns **evolution behavior**, not the definitions of those ABI mechanisms.

---

# Required implementation architecture

The repository is currently predominantly specification/documentation rather than a finished implementation tree. The existing tree contains the master documents and many specification files, while the compatibility directory already has several files in place. 

I recommend **not inventing implementation code inside the documentation directories**. When implementation begins, use a separate `src/` architecture.

## Phase 0 — independent normative foundation

These can be completed without depending on implementation code:

```text
ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md
ULABI-GOVERNANCE.md

docs/architecture/
    design-principles.md
    overview.md
    universal-model.md
    scalability.md

docs/compatibility/
    backwards-compatibility.md
    forwards-compatibility.md
    capability-discovery.md
    feature-negotiation.md
    graceful-degradation.md

The four architecture documents already exist in the repository.

Phase 1 — canonical contract definitions

docs/abi/
    core-abi.md
    calling-convention.md
    data-types.md
    memory-model.md
    stack-model.md
    register-model.md
    exception-model.md
    return-values.md

docs/type-system/
    universal-types.md
    type-descriptors.md
    generics.md
    enums.md
    structs.md
    unions.md
    type-compatibility.md

These become the definitions consumed by compatibility and interoperability implementations.

Phase 2 — interoperability

docs/interoperability/
    language-interoperability.md
    foreign-function-interface.md
    object-model.md
    name-mangling.md
    symbol-resolution.md
    cross-language-data.md

Implementation modules:

src/abi/
    interface.rs
    function.rs
    calling_convention.rs
    types.rs
    encoding.rs
    errors.rs

src/interop/
    adapter.rs
    ffi.rs
    symbols.rs
    name_mangling.rs
    object_model.rs
    data_bridge.rs

Phase 3 — compatibility engine

This is the most important code architecture arising from the document you are writing now:

src/compatibility/
    mod.rs
    version.rs
    identity.rs
    contract.rs
    classifier.rs
    backward.rs
    forward.rs
    negotiation.rs
    capability.rs
    degradation.rs
    migration.rs
    extension.rs
    unknown.rs
    compatibility_matrix.rs

Responsibilities

Module	Responsibility

version.rs	Parse and compare ULABI versions
identity.rs	Interface/type/function/extension identity
contract.rs	Canonical compatibility contract
classifier.rs	Compatible / conditional / incompatible / unknown
backward.rs	Backward compatibility rules
forward.rs	Forward compatibility rules
negotiation.rs	Feature negotiation
capability.rs	Capability discovery integration
degradation.rs	Safe fallback
migration.rs	Schema/state migration
extension.rs	Extension registration and lifecycle
unknown.rs	Unknown-field/value/operation policy
compatibility_matrix.rs	Machine-readable compatibility results


This gives each specification a corresponding implementation boundary instead of putting all compatibility logic into one giant module.


---

Phase 4 — schema and encoding implementation

src/schema/
    mod.rs
    descriptor.rs
    field.rs
    type_descriptor.rs
    version.rs
    extension.rs
    registry.rs
    validator.rs

src/encoding/
    mod.rs
    canonical.rs
    decoder.rs
    encoder.rs
    framing.rs
    limits.rs
    unknown.rs

Required schema files:

schemas/
    interface.schema.json
    type.schema.json
    function.schema.json
    error.schema.json
    capability.schema.json
    profile.schema.json
    extension.schema.json
    version.schema.json
    compatibility.schema.json
    migration.schema.json


---

Phase 5 — security boundary

src/security/
    mod.rs
    capabilities.rs
    authorization.rs
    authentication.rs
    integrity.rs
    validation.rs
    sandbox.rs
    policy.rs
    trust.rs

Compatibility MUST call into these boundaries rather than implementing its own security model.


---

Phase 6 — runtime integration

src/runtime/
    mod.rs
    interface.rs
    invocation.rs
    process.rs
    resources.rs
    lifecycle.rs
    async.rs
    cancellation.rs

The compatibility layer should determine whether a runtime can safely execute a contract; it should not become the runtime itself.


---

Phase 7 — reliability

src/reliability/
    mod.rs
    health.rs
    fault_detection.rs
    fault_isolation.rs
    recovery.rs
    rollback.rs
    self_healing.rs
    verification.rs

The self-healing system must remain policy-controlled and bounded, consistent with the architecture established in the master design.


---

Phase 8 — distributed implementation

src/distributed/
    mod.rs
    protocol.rs
    remote_call.rs
    serialization.rs
    service_discovery.rs
    errors.rs
    transport.rs
    session.rs

Transport implementations should remain adapters:

src/transport/
    mod.rs
    local.rs
    ipc.rs
    shared_memory.rs
    tcp.rs
    quic.rs
    wasm.rs

ULABI should not make TCP, QUIC, shared memory, or any other transport part of the Core.


---

Phase 9 — tooling

src/tools/
    mod.rs
    validator.rs
    compatibility_checker.rs
    schema_checker.rs
    abi_diff.rs
    conformance.rs
    diagnostics.rs

Especially important:

src/tools/abi_diff.rs

It should be able to compare:

ULABI Contract A
        |
        v
   ABI Difference
        ^
        |
ULABI Contract B

and report:

BACKWARD_COMPATIBLE
FORWARD_COMPATIBLE
CONDITIONALLY_COMPATIBLE
SAFELY_REJECTABLE
INCOMPATIBLE
UNKNOWN


---

Phase 10 — public API

Eventually:

src/
    lib.rs
    abi/
    compatibility/
    encoding/
    schema/
    security/
    runtime/
    interop/
    distributed/
    transport/
    reliability/
    tools/

The public API should expose contracts rather than implementation-specific language behavior.


---

Tests and conformance

The implementation should eventually have:

tests/
    abi/
    types/
    encoding/
    compatibility/
    security/
    runtime/
    interop/
    distributed/
    reliability/

conformance/
    core/
    types/
    memory/
    ffi/
    security/
    async/
    distributed/
    self_healing/
    compatibility/

For this specific document:

tests/compatibility/
    forward_basic.*
    forward_unknown_fields.*
    forward_unknown_enum.*
    forward_unknown_variant.*
    forward_unknown_extension.*
    forward_unknown_operation.*
    forward_unknown_error.*
    forward_security_requirement.*
    forward_resource_limits.*
    forward_preservation.*
    forward_migration.*
    forward_negotiation.*
    forward_degradation.*
    forward_downgrade.*

The language extension can eventually provide implementations in Rust, C++, Go, etc.; the tests must test the ULABI contract, not one implementation language.


---

Recommended implementation order

To satisfy your requirement that a file be finished without later needing to rewrite it because another component was added, the dependency order should be:

1. ULABI-DESIGN.md
        ↓
2. ULABI-SPEC.md
        ↓
3. ULABI-VERSIONING.md
        ↓
4. Architecture specifications
        ↓
5. ABI/type specifications
        ↓
6. Compatibility specifications
        ↓
7. Interoperability specifications
        ↓
8. Security specifications
        ↓
9. Runtime specifications
        ↓
10. Distributed specifications
        ↓
11. Schemas
        ↓
12. Reference implementation
        ↓
13. Validators
        ↓
14. Conformance suite
        ↓
15. Certification

For the compatibility family specifically:

backwards-compatibility.md
          ↓
forwards-compatibility.md
          ↓
capability-discovery.md
          ↓
feature-negotiation.md
          ↓
graceful-degradation.md
          ↓
compatibility schemas
          ↓
compatibility engine
          ↓
ABI-diff validator
          ↓
conformance tests

However, the important point is that the documents should not be edited later merely because the implementation is added. The integration contracts above are deliberately defined now.

The final code-module inventory

When implementation begins, I would lock the initial module names to:

src/
├── lib.rs
│
├── abi/
│   ├── mod.rs
│   ├── interface.rs
│   ├── function.rs
│   ├── calling_convention.rs
│   ├── types.rs
│   ├── encoding.rs
│   └── errors.rs
│
├── schema/
│   ├── mod.rs
│   ├── descriptor.rs
│   ├── field.rs
│   ├── type_descriptor.rs
│   ├── version.rs
│   ├── extension.rs
│   ├── registry.rs
│   └── validator.rs
│
├── encoding/
│   ├── mod.rs
│   ├── canonical.rs
│   ├── encoder.rs
│   ├── decoder.rs
│   ├── framing.rs
│   ├── limits.rs
│   └── unknown.rs
│
├── compatibility/
│   ├── mod.rs
│   ├── version.rs
│   ├── identity.rs
│   ├── contract.rs
│   ├── classifier.rs
│   ├── backward.rs
│   ├── forward.rs
│   ├── negotiation.rs
│   ├── capability.rs
│   ├── degradation.rs
│   ├── migration.rs
│   ├── extension.rs
│   ├── unknown.rs
│   └── compatibility_matrix.rs
│
├── interop/
│   ├── mod.rs
│   ├── adapter.rs
│   ├── ffi.rs
│   ├── symbols.rs
│   ├── name_mangling.rs
│   ├── object_model.rs
│   └── data_bridge.rs
│
├── security/
│   ├── mod.rs
│   ├── capabilities.rs
│   ├── authentication.rs
│   ├── authorization.rs
│   ├── integrity.rs
│   ├── validation.rs
│   ├── sandbox.rs
│   ├── policy.rs
│   └── trust.rs
│
├── runtime/
│   ├── mod.rs
│   ├── interface.rs
│   ├── invocation.rs
│   ├── process.rs
│   ├── resources.rs
│   ├── lifecycle.rs
│   ├── async.rs
│   └── cancellation.rs
│
├── distributed/
│   ├── mod.rs
│   ├── protocol.rs
│   ├── remote_call.rs
│   ├── serialization.rs
│   ├── service_discovery.rs
│   ├── errors.rs
│   ├── transport.rs
│   └── session.rs
│
├── transport/
│   ├── mod.rs
│   ├── local.rs
│   ├── ipc.rs
│   ├── shared_memory.rs
│   ├── tcp.rs
│   ├── quic.rs
│   └── wasm.rs
│
├── reliability/
│   ├── mod.rs
│   ├── health.rs
│   ├── fault_detection.rs
│   ├── fault_isolation.rs
│   ├── recovery.rs
│   ├── rollback.rs
│   ├── self_healing.rs
│   └── verification.rs
│
└── tools/
    ├── mod.rs
    ├── validator.rs
    ├── compatibility_checker.rs
    ├── schema_checker.rs
    ├── abi_diff.rs
    ├── conformance.rs
    └── diagnostics.rs

That is the important separation: the specification defines the contract; these modules implement the contract; tests verify the contract; adapters connect external languages and platforms.

Most importantly, nothing in this architecture makes ULABI a Zamani subsystem or a Sankofa subsystem. Both can independently implement the same ULABI contracts.