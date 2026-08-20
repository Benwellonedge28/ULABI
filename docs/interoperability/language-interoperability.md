I inspected the actual Benwellonedge28/ULABI repository before drafting this. The master architecture establishes language, runtime, platform, CPU and vendor neutrality, the Minimal Core + Profiles model, semantic types, explicit memory boundaries, transport independence, compatibility, security, and independent implementations.

I also checked the existing interoperability/ABI material. docs/interoperability/language-interoperability.md is currently only a placeholder, while cross-language-data.md and the FFI specification already establish adjacent responsibilities.

So language-interoperability.md should not duplicate those documents. Its job is to define the overall contract for interoperability between language implementations and explain how the other interoperability specifications fit together.

docs/interoperability/language-interoperability.md

Copy the following into that file:

# ULABI Language Interoperability Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: `docs/interoperability/language-interoperability.md`
Specification Family: Interoperability
Component: Language Interoperability
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Status: Normative Design Specification / Pre-Implementation
License: Apache-2.0

Normative authorities:

- `ULABI-DESIGN.md`
- `ULABI-SPEC.md`
- `ULABI-VERSIONING.md`

Related specifications:

- `docs/abi/core-abi.md`
- `docs/abi/calling-convention.md`
- `docs/abi/data-types.md`
- `docs/abi/memory-model.md`
- `docs/interoperability/foreign-function-interface.md`
- `docs/interoperability/cross-language-data.md`
- `docs/interoperability/object-model.md`
- `docs/interoperability/name-mangling.md`
- `docs/interoperability/symbol-resolution.md`
- `docs/compatibility/feature-negotiation.md`
- `docs/compatibility/capability-discovery.md`
- `docs/distributed/serialization.md`
- `docs/distributed/remote-calls.md`

---

# 1. Purpose

This specification defines the universal language-interoperability model of ULABI.

Its purpose is to allow independently implemented programming languages to interoperate through stable ULABI contracts without requiring the participating languages to share:

- syntax;
- compiler;
- compiler implementation;
- runtime;
- virtual machine;
- garbage collector;
- ownership model;
- object model;
- type system;
- exception system;
- operating system;
- processor architecture;
- hardware;
- vendor;
- implementation language.

The fundamental model is:

```text
Language A
    |
    v
Language A Adapter
    |
    v
+----------------------+
|        ULABI         |
| Universal Contract   |
+----------------------+
    ^
    |
Language B Adapter
    |
    ^
Language B

ULABI defines the contract.

Each language remains responsible for its own implementation.


---

2. Fundamental Principle

The fundamental rule is:

> ULABI interoperability is based on shared semantic contracts, not shared implementation representations.



Therefore:

Language A
    |
    | language-specific representation
    v
ULABI Semantic Contract
    |
    | language-specific representation
    v
Language B

The internal representations may be completely different.

For example:

Language A:
struct User {
    ...
}

Language B:
class User {
    ...
}

Language C:
record User {
    ...
}

These types do not become interoperable merely because they have the same name.

They become interoperable only when their ULABI mappings satisfy the same contract.


---

3. Language Neutrality

ULABI MUST remain independent of all programming languages.

The following are equally valid implementations:

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

No language may define ULABI semantics.

ULABI MUST NOT require Zamani, Sankofa, C, Rust, Python, or any other particular language.

Zamani and Sankofa remain independent projects.


---

4. Scope

This specification defines:

1. language interoperability architecture;


2. language adapters;


3. provider and consumer roles;


4. ULABI language bindings;


5. semantic contract mapping;


6. language capability declarations;


7. type-system adaptation;


8. calling-boundary adaptation;


9. memory-boundary adaptation;


10. error-boundary adaptation;


11. object-boundary adaptation;


12. symbol-boundary adaptation;


13. function identity;


14. interface identity;


15. language-specific representation isolation;


16. interoperability modes;


17. compatibility validation;


18. feature negotiation;


19. capability discovery;


20. version compatibility;


21. unsupported-feature behavior;


22. lossless and lossy adaptation;


23. ownership adaptation;


24. lifetime adaptation;


25. asynchronous interoperability;


26. callback interoperability;


27. generic and polymorphic interoperability;


28. opaque values;


29. resource handles;


30. diagnostics;


31. security boundaries;


32. failure behavior;


33. conformance requirements.



This specification does NOT redefine:

the Core ABI;

physical calling conventions;

universal primitive types;

memory-management algorithms;

serialization formats;

object-model internals;

symbol-mangling algorithms;

distributed transport;

cryptographic algorithms.


Those responsibilities belong to their respective specifications.


---

5. Architectural Position

Language interoperability is an integration layer.

ULABI
                      |
        +-------------+-------------+
        |             |             |
      Types         Calls         Errors
        |             |             |
        +-------------+-------------+
                      |
             Language Interoperability
                      |
        +-------------+-------------+
        |             |             |
       FFI      Cross-Language     Object
                 Data              Model
        |             |             |
        +-------------+-------------+
                      |
              Language Adapters
                      |
       +--------------+--------------+
       |              |              |
   Language A     Language B     Language C

The language-interoperability specification coordinates these components.

It MUST NOT absorb their individual specifications.


---

6. Interoperability Contract

Every language participating in ULABI interoperability MUST expose a language adapter.

Conceptually:

Language
    |
    v
Compiler / Runtime
    |
    v
ULABI Language Adapter
    |
    +--> Type Mapping
    +--> Call Mapping
    +--> Error Mapping
    +--> Memory Mapping
    +--> Object Mapping
    +--> Symbol Mapping
    +--> Capability Mapping
    |
    v
ULABI

The adapter is responsible for translating between the language's internal semantics and ULABI semantics.


---

7. Provider and Consumer

A language implementation may act as either:

provider;

consumer;

both.


7.1 Provider

A provider exposes ULABI interfaces.

Language
    |
    v
Implementation
    |
    v
ULABI Adapter
    |
    v
ULABI Interface

A provider MUST expose sufficient metadata for consumers to validate the interface.


---

7.2 Consumer

A consumer imports ULABI interfaces.

ULABI Interface
    |
    v
ULABI Adapter
    |
    v
Language API

The consumer MUST validate the interface before use according to the applicable security and conformance profile.


---

8. Interface Identity

A language binding MUST use stable ULABI interface identity.

It MUST NOT rely exclusively on:

source-language names;

class names;

package names;

memory addresses;

linker symbols;

filenames;

compiler-generated identifiers.


The identity hierarchy is:

ULABI Interface Identity
        |
        v
ULABI Function / Type Identity
        |
        v
Language Binding
        |
        v
Native Symbol
        |
        v
Implementation Address

Only the ULABI identity is part of the universal semantic contract.


---

9. Language Adapter

A language adapter is the implementation responsible for translating between a language and ULABI.

A conforming adapter SHOULD contain the following conceptual components:

LanguageAdapter
|
+-- InterfaceImporter
+-- InterfaceExporter
+-- TypeMapper
+-- ValueConverter
+-- CallAdapter
+-- ErrorAdapter
+-- MemoryAdapter
+-- ObjectAdapter
+-- SymbolAdapter
+-- CapabilityAdapter
+-- VersionAdapter
+-- CompatibilityValidator
+-- DiagnosticsAdapter

These are implementation modules, not necessarily separate source files in every implementation.


---

10. Type-System Independence

ULABI MUST NOT require participating languages to have identical type systems.

Languages may use:

nominal types;

structural types;

algebraic data types;

classes;

interfaces;

traits;

protocols;

records;

tuples;

unions;

tagged unions;

generics;

dynamic types;

dependent types;

ownership types;

capability types;

primitive-only systems.


Interoperability occurs through ULABI semantic types.

Language Type
      |
      v
ULABI Type Contract
      |
      v
Foreign Language Type


---

11. Type Mapping

Type mapping is governed by:

docs/interoperability/cross-language-data.md

The language-interoperability layer MUST consume those mappings.

It MUST NOT create a competing type-equivalence system.

Conceptually:

TypeMapping {
    source_type
    ulabi_type
    target_type
    conversion_policy
    ownership_policy
    lifetime_policy
    loss_policy
}

A mapping may be:

Identity
Adapted
Opaque
Unsupported

Unsupported mappings MUST fail explicitly.


---

12. Semantic Equivalence

Two language types are interoperable only if their ULABI contracts are compatible.

Example:

Language A:
uint64 UserID

Language B:
class UserID {
    BigInteger value;
}

These are interoperable only if both satisfy:

ULABI UserID Contract

Memory layout similarity alone is insufficient.


---

13. Representation Independence

ULABI MUST isolate language-specific representations.

For example:

Language A
    |
    | object
    v
ULABI Record
    |
    | object
    v
Language B

ULABI MUST NOT require:

A object layout == B object layout

Likewise:

A GC == B GC
A ownership == B ownership
A vtable == B vtable
A pointer model == B pointer model

are not requirements.


---

14. Primitive Interoperability

Primitive interoperability MUST use the ULABI primitive contracts.

Examples include:

Bool
I8
I16
I32
I64
U8
U16
U32
U64
F32
F64
Char
String
Bytes
Unit

The adapter MUST validate:

width;

signedness;

range;

representation;

encoding;

overflow behavior;

conversion semantics.


Native types such as int, long, char, or equivalent MUST NOT be treated as universally sized.


---

15. Structured Data

Structured values MUST use ULABI semantic contracts.

Supported structures include:

Record
Enum
Variant
Option
Result
List
Map
Tuple
Handle
Resource
Capability

The language may represent them differently internally.

For example:

ULABI Record
     |
     +--> C struct
     +--> Rust struct
     +--> Java record
     +--> Python object
     +--> Go struct
     +--> Swift struct
     +--> Kotlin data class

All representations remain implementation-specific.


---

16. Memory Independence

ULABI MUST NOT require a common memory-management system.

Participating languages may use:

manual memory management;

reference counting;

tracing garbage collection;

ownership and borrowing;

arenas;

regions;

immutable memory;

custom allocators.


Interoperability occurs at the ULABI memory boundary.

Memory semantics are governed by:

docs/abi/memory-model.md

and related memory specifications.


---

17. Ownership Adaptation

An adapter MUST preserve ULABI ownership semantics.

Possible boundary semantics include:

Borrow
Shared
Owned
Move
Transfer
Consume
ImmutableShared

If a language cannot safely represent a required ownership transition, the adapter MUST:

1. use an explicitly permitted safe adaptation; or


2. use an opaque handle; or


3. make a permitted copy; or


4. reject the operation.



Unsafe implicit ownership conversion is prohibited.


---

18. Lifetime Adaptation

A language adapter MUST preserve the declared ULABI lifetime.

Possible lifetimes include:

DuringCall
UntilReturn
UntilRelease
Pinned
Explicit
InterfaceLifetime

A target language MUST NOT retain a value after the permitted lifetime.

If the target language cannot enforce the required lifetime, the adapter MUST use a safe strategy or reject the binding.


---

19. Calling Interoperability

Calling semantics are governed by:

docs/abi/calling-convention.md

The language-interoperability layer MUST NOT define another calling convention.

The relationship is:

Language
    |
    v
Language Call
    |
    v
ULABI Call Contract
    |
    v
Native ABI / Runtime

The adapter performs the necessary lowering.


---

20. Error Interoperability

Languages may represent errors through:

exceptions;

result types;

status codes;

error objects;

tagged unions;

traps;

conditions;

checked errors.


These MUST map to the ULABI error contract.

For example:

Language Exception
        |
        v
ULABI Error
        |
        v
Target Language Error

A private language-runtime exception object MUST NOT automatically cross the boundary.


---

21. Panic and Fatal Failure

The adapter MUST distinguish normal errors from fatal failures.

The following MUST NOT automatically be treated as ordinary application errors:

process termination;

segmentation failure;

illegal instruction;

runtime corruption;

fatal memory failure;

unrecoverable runtime panic.


The appropriate reliability and error specifications determine handling.


---

22. Object Interoperability

Object interoperability is delegated to:

docs/interoperability/object-model.md

ULABI MUST NOT require languages to share:

inheritance;

virtual dispatch;

vtables;

class layouts;

reflection systems;

method tables;

garbage-collector metadata.


When direct object interoperability is unsafe, implementations SHOULD use:

Opaque Handle

or a ULABI semantic data representation.


---

23. Symbol Interoperability

Symbol naming and resolution are delegated to:

docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md

The language-interoperability layer consumes these mechanisms.

A native symbol MUST NOT become the semantic identity of a ULABI interface.


---

24. FFI Integration

The FFI boundary is defined by:

docs/interoperability/foreign-function-interface.md

The relationship is:

Language Interoperability
          |
          v
          FFI
          |
    +-----+-----+
    |     |     |
 Types  Calls  Errors
    |     |     |
    +-----+-----+
          |
        ULABI

The FFI provides the language-specific binding mechanism.

Language interoperability defines the overall semantic model.


---

25. Cross-Language Data Integration

Cross-language data semantics are defined by:

docs/interoperability/cross-language-data.md

This includes:

primitive mappings;

structured values;

collections;

strings;

binary data;

options;

results;

ownership;

lifetimes;

conversion;

loss detection;

opaque values.


The language-interoperability layer MUST consume these contracts.


---

26. Generic and Polymorphic Types

Languages may implement generics differently.

ULABI MUST NOT require a universal generic implementation strategy.

A generic interface MUST expose sufficient semantic information to determine:

required type parameters;

constraints;

variance where applicable;

instantiated contract;

representation requirements;

compatibility.


If generic interoperability cannot be established safely, the adapter MUST reject it or expose a specialized ULABI interface.


---

27. Dynamic Languages

Dynamically typed languages MAY participate in ULABI.

A dynamic language adapter MUST validate runtime values against the ULABI contract.

For example:

Dynamic Value
     |
     v
Runtime Validation
     |
     v
ULABI Type

Dynamic typing MUST NOT eliminate ULABI contract validation.


---

28. Static Languages

Statically typed languages SHOULD validate ULABI contracts at compile time whenever possible.

Runtime validation MAY still be required for:

dynamically loaded modules;

remote interfaces;

negotiated interfaces;

untrusted implementations;

optional profiles.



---

29. Capability Interoperability

Language capabilities MUST map to ULABI capabilities.

Capabilities are governed by:

docs/security/capability-security.md

and:

docs/compatibility/capability-discovery.md

A language adapter MUST NOT silently grant capabilities merely because the underlying language can perform an operation.

Example:

Language can access filesystem
        !=
ULABI interface has filesystem capability

Capabilities MUST be explicitly granted.


---

30. Effect Interoperability

ULABI effects such as:

Pure
ReadsMemory
WritesMemory
Network
Filesystem
GPU
Process
Time
Randomness
ExternalDevice
NonDeterministic

MUST remain explicit.

A language adapter MUST NOT remove meaningful effects from an imported contract.


---

31. Synchronous and Asynchronous Interoperability

ULABI distinguishes:

Synchronous
Asynchronous
Blocking
NonBlocking
Streaming
LongRunning
Cancellable

A language adapter MUST preserve these semantics.

A synchronous function MUST NOT silently become a network-dependent asynchronous operation.

An asynchronous function MUST NOT be presented as synchronous if doing so changes its semantics.


---

32. Callback Interoperability

Callbacks MUST use stable ULABI function identities.

Conceptually:

Language A
   |
   | callback registration
   v
ULABI Callback
   |
   v
Language B

Callback contracts MUST specify:

function identity;

parameter types;

result type;

lifetime;

ownership;

capabilities;

execution context;

cancellation behavior;

failure behavior.


A callback MUST NOT outlive its declared lifetime.


---

33. Function References

Function references MUST be represented through ULABI function identity or an opaque function handle.

A raw machine address MUST NOT be treated as a portable function identity.


---

34. Resource Interoperability

Resources that cannot safely cross language boundaries as ordinary values SHOULD use ULABI handles.

Examples:

FileHandle
SocketHandle
GPUResource
DeviceHandle
DatabaseConnection
WindowHandle
ProcessHandle

Handles MUST have explicit:

identity;

ownership;

lifetime;

capability requirements;

release semantics.



---

35. Zero-Copy Interoperability

Zero-copy operation MAY be used when:

1. memory ownership is compatible;


2. lifetime is enforceable;


3. alignment requirements are satisfied;


4. mutation semantics are compatible;


5. security requirements are satisfied;


6. representation is compatible;


7. the receiving language cannot violate the source contract.



Otherwise the adapter MUST copy or use an opaque representation.

Performance MUST NOT override memory safety.


---

36. Streaming Interoperability

Large data SHOULD support streaming when the applicable profile is enabled.

Example:

Language A
    |
    v
ULABI Stream<T>
    |
    v
Language B

The adapter MUST preserve:

ordering;

backpressure;

cancellation;

ownership;

lifetime;

error semantics;

completion semantics.



---

37. Compatibility

Language interoperability MUST use the ULABI compatibility system.

Relevant specifications include:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

A language implementation MUST NOT claim interoperability merely because a function name and parameter count match.


---

38. Feature Negotiation

When an implementation supports only a subset of a ULABI interface, feature negotiation MUST determine whether interoperability is possible.

Conceptually:

Provider Capabilities
        |
        v
Feature Negotiation
        |
        v
Consumer Requirements
        |
        v
Compatible Contract?
      /     \
    YES      NO
     |        |
    Bind     Reject

Negotiation MUST NOT silently weaken security or safety requirements.


---

39. Unsupported Features

If a language cannot represent a required ULABI feature, the implementation MUST explicitly report:

Unsupported

Possible responses include:

Reject
Fallback
Copy
OpaqueHandle
SpecializedBinding
NegotiatedSubset

The fallback MUST preserve semantics.


---

40. Lossy Adaptation

Lossy conversion is permitted only when explicitly authorized.

Examples include:

F64 -> F32
I64 -> I32
HighPrecisionDecimal -> Float
RichObject -> ReducedRecord

The adapter MUST identify the loss.

Silent semantic degradation is prohibited.


---

41. Security Boundary

Language interoperability is a security boundary.

The adapter MUST validate:

interface identity;

version;

signature;

types;

capabilities;

ownership;

lifetime;

effects;

integrity;

required security profile.


A language runtime MUST NOT be trusted merely because it successfully loaded a module.


---

42. Untrusted Languages and Modules

A ULABI implementation MAY interact with an untrusted implementation.

Where applicable, the implementation SHOULD use:

process isolation;

sandboxing;

capability restriction;

memory isolation;

validation;

cryptographic integrity;

resource limits.


Language interoperability MUST NOT imply mutual trust.


---

43. Failure Model

Failures MUST be explicit.

Possible interoperability failures include:

InterfaceNotFound
VersionMismatch
SignatureMismatch
TypeMismatch
OwnershipMismatch
LifetimeMismatch
CapabilityDenied
EffectMismatch
UnsupportedFeature
InvalidValue
ConversionFailure
SecurityValidationFailure
IntegrityFailure
RuntimeFailure

Failures MUST NOT be silently converted into successful operations.


---

44. Adapter Failure

If a language adapter fails while translating a value or call:

Language
   |
   v
Adapter
   |
   X
Failure
   |
   v
ULABI Error

The adapter MUST report a ULABI-compatible error whenever the failure is representable as a recoverable contract failure.


---

45. No Semantic Guessing

A conforming implementation MUST NOT guess semantics.

It MUST NOT infer:

ownership;

lifetime;

mutability;

type identity;

field identity;

function identity;

capability;

effect;

encoding;

error semantics.


from insufficient metadata.

When the contract is ambiguous, the safe behavior is rejection.


---

46. Language Adapter Isolation

A language adapter MUST isolate language-specific implementation details.

For example:

+--------------------------------------+
| Language A                           |
|                                      |
| compiler                             |
| runtime                              |
| GC / ownership / object model        |
+------------------+-------------------+
                   |
                   v
             Language Adapter
                   |
                   v
+--------------------------------------+
| ULABI                                |
| Stable semantic contract             |
+--------------------------------------+

ULABI MUST NOT expose private runtime internals unless an explicit extension profile defines them.


---

47. Interoperability Modes

ULABI language interoperability MAY operate:

47.1 In-process

Language A
    |
   ULABI
    |
Language B

47.2 Out-of-process

Process A
Language A
    |
   ULABI
    |
Process B
Language B

47.3 Distributed

Machine A
Language A
    |
   ULABI
    |
 Transport
    |
   ULABI
    |
Machine B
Language B

The semantic contract may remain stable, but execution semantics MUST remain explicit.


---

48. Locality

A language adapter MUST preserve locality semantics.

ULABI distinguishes:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

A local function MUST NOT silently become a remote operation.


---

49. Versioning

Every language binding MUST identify:

ULABI Core Version
Interface Version
Function Version
Required Profiles
Required Features

Versioning follows:

ULABI-VERSIONING.md

and the compatibility specifications.


---

50. Determinism

Where an interface declares deterministic behavior, language adapters MUST preserve it.

An adapter MUST NOT introduce nondeterminism through:

hidden network access;

uncontrolled concurrency;

implicit randomness;

uncontrolled time;

unspecified iteration order.


If nondeterminism is unavoidable, it MUST be represented by the contract.


---

51. Diagnostics

Language adapters SHOULD expose diagnostics including:

interface identity
function identity
source language
target language
source type
ULABI type
target type
conversion
version
profiles
capabilities
failure reason

Diagnostics MUST NOT expose sensitive information unless permitted by the applicable security profile.


---

52. Conformance Requirements

A conforming language adapter MUST:

1. implement ULABI interface identity;


2. implement contract validation;


3. use ULABI semantic type mappings;


4. preserve ownership semantics;


5. preserve lifetime semantics;


6. preserve error semantics;


7. preserve capability requirements;


8. preserve effect declarations;


9. respect version compatibility;


10. reject incompatible bindings;


11. avoid unsafe implicit conversions;


12. preserve stable function identity;


13. correctly integrate with the ULABI FFI;


14. correctly handle unsupported features;


15. provide required diagnostics;


16. pass applicable interoperability tests.




---

53. Conformance Levels

Implementations MAY declare:

ULABI Language Interoperability - Basic
ULABI Language Interoperability - Standard
ULABI Language Interoperability - Advanced
ULABI Language Interoperability - Certified

The exact requirements of these levels are defined by:

docs/standards/compliance-levels.md

and:

docs/standards/conformance.md


---

54. Required Conformance Tests

The language interoperability test suite MUST eventually include:

Identity

interface identity;

function identity;

type identity.


Types

primitive types;

records;

enums;

variants;

options;

results;

collections;

opaque handles.


Conversion

lossless conversion;

lossy conversion;

overflow;

underflow;

invalid encoding;

unsupported conversion.


Memory

ownership transfer;

borrowing;

lifetime enforcement;

mutation;

immutable sharing.


Calls

synchronous calls;

asynchronous calls;

callbacks;

cancellation;

function references.


Errors

normal errors;

exceptions;

cancellation;

timeout;

capability denial;

contract violation.


Compatibility

version negotiation;

feature negotiation;

unsupported feature;

forward compatibility;

backward compatibility.


Security

unauthorized capability;

invalid interface;

malicious metadata;

invalid type;

malformed value;

invalid handle.



---

55. Reference Architecture

A reference language adapter SHOULD have this conceptual architecture:

reference/
|
+-- language_adapter/
|   |
|   +-- interface_importer
|   +-- interface_exporter
|   +-- type_mapper
|   +-- value_converter
|   +-- call_adapter
|   +-- error_adapter
|   +-- memory_adapter
|   +-- object_adapter
|   +-- symbol_adapter
|   +-- capability_adapter
|   +-- version_adapter
|   +-- validator
|   +-- diagnostics
|
+-- tests/
|   |
|   +-- types
|   +-- calls
|   +-- memory
|   +-- errors
|   +-- compatibility
|   +-- security
|   +-- callbacks
|   +-- async

The reference implementation MUST NOT become the normative definition of ULABI.

The specification remains authoritative.


---

56. Required Invariants

The following invariants are mandatory.

Invariant 1 — Language Independence

No language defines ULABI semantics.

Invariant 2 — Semantic Identity

Equivalent values MUST be identified through ULABI contracts.

Invariant 3 — No Unsafe Guessing

Missing semantic information MUST result in rejection or an explicitly defined safe fallback.

Invariant 4 — Memory Safety

Language adaptation MUST NOT violate ownership or lifetime contracts.

Invariant 5 — Capability Preservation

Adapters MUST NOT silently expand capabilities.

Invariant 6 — Error Preservation

Errors MUST NOT silently become successful results.

Invariant 7 — Version Safety

Incompatible interfaces MUST NOT be bound.

Invariant 8 — Representation Independence

Language-specific representations MUST remain implementation details.

Invariant 9 — Locality Preservation

Local execution MUST NOT silently become remote execution.

Invariant 10 — Independent Implementability

Two organizations MUST be able to implement language interoperability independently from this specification.


---

57. Required Integration Contract

This document integrates with the repository as follows:

ULABI-DESIGN.md
                                |
                                v
                          ULABI-SPEC.md
                                |
                                v
                        Language Interoperability
                                |
          +---------------------+----------------------+
          |                     |                      |
          v                     v                      v
   Cross-Language Data         FFI              Object Model
          |                     |                      |
          v                     v                      v
       Types                  Calls                 Objects
          |                     |                      |
          +---------------------+----------------------+
                                |
                +---------------+---------------+
                |               |               |
                v               v               v
          Name Mangling   Symbol Resolution   Compatibility
                                                |
                                                v
                                      Feature Negotiation
                                                |
                                                v
                                      Capability Discovery

No downstream document needs to redefine the language-interoperability architecture.


---

58. Integration Ownership

The responsibilities are intentionally separated.

Concern	Authoritative document

Overall architecture	ULABI-DESIGN.md
Normative Core rules	ULABI-SPEC.md
Core ABI	docs/abi/core-abi.md
Calling semantics	docs/abi/calling-convention.md
Data types	docs/abi/data-types.md
Memory boundaries	docs/abi/memory-model.md
Language interoperability	this document
Cross-language data	docs/interoperability/cross-language-data.md
FFI	docs/interoperability/foreign-function-interface.md
Object interoperability	docs/interoperability/object-model.md
Name mapping	docs/interoperability/name-mangling.md
Symbol lookup	docs/interoperability/symbol-resolution.md
Version compatibility	ULABI-VERSIONING.md
Feature negotiation	docs/compatibility/feature-negotiation.md
Capability discovery	docs/compatibility/capability-discovery.md
Serialization	docs/distributed/serialization.md
Remote calls	docs/distributed/remote-calls.md
Conformance	docs/standards/conformance.md
Tests	docs/standards/test-suite.md


This ownership model prevents circular rewriting.


---

59. Implementation Rule

An implementation MUST be able to complete its language adapter without modifying this specification merely because another language adapter has been implemented.

Each adapter is independent:

C Adapter       ──┐
Rust Adapter     ──┤
Python Adapter   ──┤
Go Adapter       ──┤
Java Adapter     ──┤
Swift Adapter    ──┤
Kotlin Adapter   ──┤
Fortran Adapter  ──┤
Ada Adapter      ──┤
Zamani Adapter   ──┤
Sankofa Adapter  ──┤
Future Adapter   ──┘
                  |
                  v
                ULABI

No adapter is dependent on another language adapter.


---

60. Security Requirements

A conforming implementation MUST:

validate interfaces before unsafe invocation;

validate type contracts;

validate capabilities;

preserve ownership;

preserve lifetimes;

reject incompatible contracts;

prevent unauthorized resource access;

prevent unsafe pointer exposure;

prevent runtime metadata leakage;

validate dynamically loaded interfaces;

respect sandbox boundaries where applicable.



---

61. Failure-Oriented Design

Language interoperability MUST assume that adapters can fail.

The preferred model is:

Input
  |
  v
Validate
  |
  +---- invalid ----> Reject
  |
 valid
  |
  v
Convert
  |
  +---- failure ----> ULABI Error
  |
 success
  |
  v
Invoke
  |
  +---- failure ----> ULABI Error / Failure
  |
 success
  |
  v
Validate Result
  |
  +---- invalid ----> Reject Result
  |
 valid
  |
  v
Return

An implementation MUST NOT continue execution using an invalidly adapted value.


---

62. What This Specification Does Not Permit

ULABI language interoperability MUST NOT become:

a universal programming language;

a universal object model;

a universal runtime;

a universal garbage collector;

a universal memory manager;

a requirement that all languages compile to one intermediate language;

a requirement that all languages use one ABI internally;

a requirement that all languages share one compiler;

a requirement that all languages share one virtual machine.


ULABI remains a boundary contract.


---

63. Final Architectural Rule

The definitive rule is:

> ULABI defines how independently implemented languages interoperate; it does not define how those languages are implemented.



Therefore:

Language A
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
Language B

Both languages remain independent.

Both implementations may evolve independently.

Only the agreed ULABI contract must remain compatible.

## What should be done under which files

The important point from the repository inspection is that **we should not create another giant implementation-specific language layer**. The documentation should be completed in dependency order.

### Phase 1 — independent contract

**1. `docs/interoperability/language-interoperability.md` — this file**

This is the independent architectural contract.

It owns:

- language adapter model;
- semantic interoperability;
- provider/consumer model;
- representation independence;
- language capability model;
- interoperability modes;
- integration ownership;
- failure rules;
- conformance boundary.

It consumes the other specifications rather than redefining them.

This is the file you can consider complete independently after copying the specification above.

---

### Phase 2 — semantic data contract

**2. `docs/interoperability/cross-language-data.md`**

The repository already has a substantial draft here. It should own:

- type mappings;
- conversion;
- lossless/lossy conversion;
- records;
- enums;
- variants;
- collections;
- strings;
- bytes;
- options/results;
- ownership at data boundaries;
- opaque values;
- zero-copy data;
- streaming data.

It should **not** redefine the overall language-adapter architecture.

The existing draft already explicitly separates these responsibilities from `data-types.md`, `core-abi.md`, `memory-model.md`, serialization, and distributed invocation. 

---

### Phase 3 — invocation contract

**3. `docs/interoperability/foreign-function-interface.md`**

This already has a substantial specification rather than being a simple placeholder. It should own:

- imports;
- exports;
- function binding;
- interface binding;
- signature validation;
- FFI lifecycle;
- dynamic/static binding;
- callbacks;
- function references;
- foreign values;
- FFI failure.

It consumes:

```text
language-interoperability.md
cross-language-data.md
core-abi.md
calling-convention.md
memory-model.md

The existing FFI draft explicitly establishes this separation and says it should not redefine the Core ABI, calling convention, data types, memory model, serialization, object model, name mangling, or symbol resolution.


---

Phase 4 — specialized interoperability contracts

These should then be completed independently:

docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md

Their ownership is:

object-model.md

object identity;

method identity;

interfaces/protocols;

inheritance adaptation;

virtual dispatch;

opaque objects;

object lifetime.


name-mangling.md

language symbol → ULABI identity;

deterministic naming;

collision prevention;

source-language names;

ABI names.


symbol-resolution.md

locating implementations;

static symbols;

dynamic symbols;

interface discovery;

resolution failure;

symbol validation.


None should redefine language interoperability.


---

Required implementation code/modules

ULABI currently appears primarily specification-oriented, so these are required implementation modules to build, not a claim that all already exist in the repository.

I would use language-neutral conceptual modules first:

implementations/
└── core/
    ├── interface/
    │   ├── interface_identity
    │   ├── function_identity
    │   ├── type_identity
    │   └── contract_metadata
    │
    ├── language/
    │   ├── language_adapter
    │   ├── language_descriptor
    │   ├── language_capabilities
    │   └── language_profile
    │
    ├── types/
    │   ├── type_mapper
    │   ├── type_descriptor
    │   ├── value_converter
    │   ├── conversion_policy
    │   └── compatibility_checker
    │
    ├── calls/
    │   ├── call_adapter
    │   ├── argument_adapter
    │   ├── result_adapter
    │   ├── callback_adapter
    │   └── function_reference
    │
    ├── memory/
    │   ├── ownership_adapter
    │   ├── lifetime_adapter
    │   ├── borrow_adapter
    │   ├── shared_value
    │   └── opaque_handle
    │
    ├── errors/
    │   ├── error_adapter
    │   ├── exception_adapter
    │   ├── panic_adapter
    │   └── failure_mapper
    │
    ├── objects/
    │   ├── object_adapter
    │   ├── method_adapter
    │   └── object_handle
    │
    ├── symbols/
    │   ├── name_mapper
    │   ├── symbol_resolver
    │   └── symbol_validator
    │
    ├── capabilities/
    │   ├── capability_adapter
    │   ├── capability_validator
    │   └── effect_validator
    │
    ├── compatibility/
    │   ├── version_validator
    │   ├── feature_negotiator
    │   └── capability_discovery
    │
    ├── validation/
    │   ├── interface_validator
    │   ├── signature_validator
    │   ├── type_validator
    │   ├── contract_validator
    │   └── security_validator
    │
    └── diagnostics/
        ├── interoperability_diagnostics
        ├── conversion_diagnostics
        └── failure_diagnostics

Then language-specific adapters should be completely independent:

implementations/
└── languages/
    ├── c/
    │   └── ulabi_adapter
    ├── cpp/
    │   └── ulabi_adapter
    ├── rust/
    │   └── ulabi_adapter
    ├── go/
    │   └── ulabi_adapter
    ├── java/
    │   └── ulabi_adapter
    ├── python/
    │   └── ulabi_adapter
    ├── swift/
    │   └── ulabi_adapter
    ├── kotlin/
    │   └── ulabi_adapter
    ├── fortran/
    │   └── ulabi_adapter
    ├── ada/
    │   └── ulabi_adapter
    └── future/
        └── ulabi_adapter

Zamani and Sankofa should be optional external implementations, not dependencies of ULABI.


---

Tests/modules that should accompany them

The implementation should eventually have:

tests/
├── interoperability/
│   ├── interface_identity
│   ├── function_identity
│   ├── primitive_types
│   ├── structured_types
│   ├── conversions
│   ├── ownership
│   ├── lifetimes
│   ├── errors
│   ├── callbacks
│   ├── async
│   ├── capabilities
│   ├── compatibility
│   ├── unsupported_features
│   ├── security
│   └── diagnostics
│
└── languages/
    ├── c
    ├── cpp
    ├── rust
    ├── go
    ├── java
    ├── python
    ├── swift
    ├── kotlin
    ├── fortran
    └── ada

And ultimately:

conformance/
└── language-interoperability/
    ├── identity/
    ├── types/
    ├── conversion/
    ├── memory/
    ├── calls/
    ├── errors/
    ├── callbacks/
    ├── async/
    ├── security/
    └── compatibility/

This gives us the important property you requested: once language-interoperability.md is finished, later work on FFI, cross-language data, object models, or individual language adapters should implement its already-defined contracts rather than forcing this document to be rewritten.

The repository's current architecture already supports that separation: the master design establishes the universal contract and explicitly calls for a minimal stable Core with extension profiles, rather than tying ULABI to one implementation.