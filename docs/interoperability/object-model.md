I inspected the actual Benwellonedge28/ULABI repository before drafting this. The master architecture explicitly establishes language/runtime/platform/architecture/vendor neutrality, the Minimal Core + Profiles model, semantic contracts, explicit memory boundaries, transport independence, compatibility, security, and independent implementations.

I also checked the existing interoperability documents. docs/interoperability/object-model.md is currently only a placeholder, so this is a good independent specification to complete now.  The repository already separates the responsibilities of language interoperability, FFI, cross-language data, name mangling, symbol resolution, ABI calling/data/memory models, serialization, and compatibility.

1. Correct responsibility of object-model.md

The important boundary is:

language-interoperability.md
        │
        ├── FFI
        ├── cross-language-data
        ├── object-model  ← THIS DOCUMENT
        ├── name-mangling
        └── symbol-resolution

object-model.md should not define a universal object-oriented programming model.

It should define how an implementation that has objects/classes/instances/interfaces/traits/protocols/etc. can expose object-like behavior through a language-neutral ULABI contract.

It therefore must not assume:

classes;

inheritance;

vtables;

garbage collection;

reference counting;

virtual methods;

reflection;

a particular pointer layout;

a particular language;

Zamani;

Sankofa.


That is especially important because the existing language-interoperability design already says ULABI must not require languages to share an object model, vtable, class layout, reflection system, or GC metadata.


---

2. docs/interoperability/object-model.md

Copy the following as the complete file.

# ULABI Object Model Interoperability Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Document: `docs/interoperability/object-model.md`

Specification Family: Interoperability
Component: Object Model Interoperability

Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft

Status: Normative Design Specification / Pre-Implementation

License: Apache-2.0

Normative Authorities:

- `ULABI-DESIGN.md`
- `ULABI-SPEC.md`
- `ULABI-VERSIONING.md`

Related Specifications:

- `docs/architecture/universal-model.md`
- `docs/abi/core-abi.md`
- `docs/abi/calling-convention.md`
- `docs/abi/data-types.md`
- `docs/abi/memory-model.md`
- `docs/abi/exception-model.md`
- `docs/interoperability/language-interoperability.md`
- `docs/interoperability/foreign-function-interface.md`
- `docs/interoperability/cross-language-data.md`
- `docs/interoperability/name-mangling.md`
- `docs/interoperability/symbol-resolution.md`
- `docs/runtime/runtime-interface.md`
- `docs/compatibility/feature-negotiation.md`
- `docs/compatibility/capability-discovery.md`
- `docs/distributed/serialization.md`
- `docs/distributed/remote-calls.md`
- `docs/security/security-model.md`
- `docs/security/capability-security.md`
- `docs/standards/conformance.md`
- `docs/standards/test-suite.md`

---

# 1. Purpose

This specification defines how object-oriented, object-like, component-oriented, and encapsulated data structures can interoperate through ULABI.

ULABI MUST NOT define a universal programming-language object model.

Instead, this specification defines a language-neutral boundary model through which independently implemented systems may expose:

- objects;
- instances;
- methods;
- interfaces;
- properties;
- resources;
- capabilities;
- opaque values;
- lifecycle operations;
- identity;
- state;
- behavior.

The objective is to permit interoperability between languages that use fundamentally different programming models.

For example:

```text
Language A
    |
    | class
    v
ULABI Object Contract
    |
    | record / interface / handle
    v
Language B

or:

Language A
    |
    | struct + functions
    v
ULABI Object Contract
    |
    | trait implementation
    v
Language B

or:

Language A
    |
    | actor
    v
ULABI Object Contract
    |
    | service proxy
    v
Language B

All of these are potentially valid.

The object model is therefore a boundary contract, not a requirement that participating languages use the same internal object representation.


---

2. Fundamental Principle

The fundamental rule is:

> ULABI object interoperability is based on semantic contracts, stable identities, explicit capabilities, and defined lifecycle behavior rather than shared object layout.



Therefore:

Language Object
      |
      v
Language Adapter
      |
      v
ULABI Object Contract
      |
      v
Language Adapter
      |
      v
Foreign Object Representation

The following MUST NOT be assumed to be equivalent:

A class == B class
A object layout == B object layout
A vtable == B vtable
A inheritance model == B inheritance model
A GC == B GC
A pointer == B pointer
A method table == B method table

Two implementations are interoperable when they satisfy the same ULABI contract.


---

3. Scope

This specification defines:

1. object identity;


2. object contracts;


3. object instances;


4. object state;


5. object behavior;


6. methods;


7. method identity;


8. interfaces;


9. interface implementation;


10. properties;


11. fields;


12. encapsulation;


13. opaque objects;


14. object handles;


15. object references;


16. object capabilities;


17. ownership;


18. borrowing;


19. lifetime;


20. object creation;


21. object destruction;


22. resource release;


23. inheritance adaptation;


24. composition;


25. delegation;


26. polymorphism;


27. dynamic dispatch;


28. static dispatch;


29. interface dispatch;


30. method dispatch;


31. callbacks;


32. events;


33. asynchronous objects;


34. actor-like objects;


35. proxy objects;


36. remote object references;


37. object serialization boundaries;


38. object identity preservation;


39. object equality;


40. hashing;


41. introspection;


42. reflection boundaries;


43. generic objects;


44. versioning;


45. compatibility;


46. security;


47. failure behavior;


48. lifecycle validation;


49. conformance.



This specification does NOT define:

a universal programming language;

a universal class syntax;

a universal inheritance system;

a universal garbage collector;

a universal memory allocator;

a universal pointer representation;

a universal vtable;

a universal serialization format;

a universal RPC protocol;

a universal reflection API.


Those responsibilities belong to other ULABI specifications or implementation-specific systems.


---

4. Architectural Position

The object model is positioned between language-specific object systems and ULABI semantic contracts.

ULABI
                      |
          +-----------+-----------+
          |           |           |
        Types       Calls       Errors
          |           |           |
          +-----------+-----------+
                      |
              Object Interoperability
                      |
       +--------------+--------------+
       |              |              |
     Objects      Interfaces       Handles
       |              |              |
       +--------------+--------------+
                      |
               Language Adapter
                      |
       +--------------+--------------+
       |              |              |
   Language A     Language B     Language C

The object model therefore integrates with, but does not replace:

the Core ABI;

the calling convention;

the universal type system;

the memory model;

FFI;

cross-language data;

symbol resolution;

security;

compatibility.



---

5. Object Contract

A ULABI object MUST be represented through an explicit semantic contract.

Conceptually:

ObjectContract {
    object_type_id
    version
    interfaces
    state_contract
    method_contracts
    lifecycle_contract
    ownership_contract
    capability_requirements
    effect_contract
    execution_contract
    compatibility_policy
}

The exact binary representation is implementation-defined unless standardized by another ULABI specification.

The semantic fields above MUST be represented by the applicable ULABI metadata mechanism.


---

6. Object Type Identity

Every externally visible object type MUST have a stable ULABI identity.

The identity MUST NOT depend solely upon:

source-language class name;

package name;

module name;

compiler-generated symbol;

memory address;

vtable address;

pointer value.


For example:

org.example.storage/DatabaseConnection

may identify a semantic object contract.

A C implementation, Rust implementation, Java implementation, or another implementation MAY all implement that contract.


---

7. Object Instance Identity

An object type identity identifies the contract.

An object instance identity identifies a particular instance.

These are distinct:

Object Type Identity
        |
        +---- Object Instance A
        |
        +---- Object Instance B
        |
        +---- Object Instance C

An object instance identity MUST NOT be assumed to be the same as:

a memory address;

a pointer;

an allocator address;

an object header;

a runtime object ID.


A local implementation MAY use those mechanisms internally.

The ULABI-visible identity MUST follow the applicable identity contract.


---

8. Object Identity Stability

Object identity MUST remain stable for the lifetime defined by the contract.

If an object is moved in memory:

Address A
   |
   v
Object
   |
   | move
   v
Address B

its ULABI identity MUST remain unchanged if the contract promises stable identity.

This allows implementations to use:

moving garbage collectors;

compacting heaps;

arenas;

relocation;

memory pools;

remote proxies.



---

9. Object Representation Independence

ULABI MUST NOT expose private implementation layout unless an explicit low-level profile requires it.

A language may internally use:

Object Header
VTable
Fields
GC Metadata

Another may use:

Struct
Function Table
Reference Counter

Another may use:

Handle
Runtime Registry
Method Dispatch Table

These are all implementation details.

The ULABI object contract is the authoritative boundary.


---

10. Object State

Object state consists of data governed by the object contract.

State MAY be represented internally as:

fields;

properties;

closures;

external storage;

database records;

handles;

actor state;

device state;

remote state.


The internal representation is not part of the universal contract unless explicitly exposed.


---

11. State Visibility

An object contract MUST explicitly identify whether state is:

public;

read-only;

mutable;

write-only;

private;

protected by methods;

capability-protected;

opaque;

unavailable.


Private state MUST NOT become externally accessible merely because an implementation happens to store it in a visible memory structure.


---

12. Encapsulation

ULABI object contracts SHOULD preserve encapsulation.

A consumer SHOULD interact with an object through:

Interface
   |
   +-- Method
   +-- Property
   +-- Capability
   +-- Lifecycle Operation

rather than directly manipulating implementation memory.

Direct memory exposure MUST require an explicit ULABI profile.


---

13. Methods

A method is an operation associated with an object contract.

Conceptually:

Method {
    method_id
    parameters
    result
    errors
    ownership
    lifetime
    effects
    capabilities
    execution_semantics
}

The method identity MUST be stable.

Method naming is governed by:

docs/interoperability/name-mangling.md

Method lookup is governed by:

docs/interoperability/symbol-resolution.md

Method invocation is governed by:

docs/abi/calling-convention.md


---

14. Method Identity

A method MUST be identified by:

Object / Interface Identity
        +
Method Identity
        +
Contract Identity

A source-language method name alone is insufficient.

For example:

A.read()
B.read()

MUST remain distinguishable when they represent different contracts.


---

15. Method Signature

A method signature MUST include all semantically relevant properties required by the Core ABI.

At minimum:

MethodIdentity
Parameters
ParameterTypes
PassingModes
Ownership
Lifetime
ResultType
ErrorContract
ExecutionMode
Effects
Capabilities
Version

A consumer MUST validate the contract before invoking a method.


---

16. Method Dispatch

ULABI supports several semantic dispatch models:

StaticDispatch
DynamicDispatch
InterfaceDispatch
CapabilityDispatch
ProxyDispatch
RemoteDispatch

The dispatch mechanism is implementation-defined.

The semantic method identity MUST remain stable across dispatch mechanisms.


---

17. Static Dispatch

An implementation MAY resolve a method at compile time.

Example:

Consumer
   |
   v
Known Interface
   |
   v
Static Target

This is valid when the implementation can prove the required contract.


---

18. Dynamic Dispatch

An implementation MAY resolve a method at runtime.

Example:

Object
  |
  v
Interface Table
  |
  v
Method Identity
  |
  v
Implementation

The implementation MUST validate that the selected method satisfies the ULABI contract.


---

19. Interface Dispatch

ULABI strongly favors explicit interfaces for portable object interoperability.

An interface defines a set of externally visible capabilities.

Example:

Readable {
    read(...)
}

Writable {
    write(...)
}

A language MAY implement these using:

interfaces;

traits;

protocols;

abstract classes;

function tables;

structs containing function pointers;

closures;

generated adapters.


The implementation mechanism is not normative.


---

20. Interface Identity

An interface MUST have a stable identity.

Example:

org.example.io/Readable

An interface identity MUST NOT depend solely on:

source-language name;

package;

class name;

module path.



---

21. Interface Implementation

An object implements an interface when it satisfies every required contract.

Conceptually:

Object
  |
  +-- implements --> Interface A
  |
  +-- implements --> Interface B

The implementation MUST satisfy:

required methods;

method signatures;

ownership rules;

lifetime rules;

error behavior;

capability requirements;

effect declarations;

compatibility requirements.



---

22. Structural and Nominal Interfaces

ULABI MAY support both:

Nominal Interface

and:

Structural Interface

A nominal implementation explicitly declares:

Object implements InterfaceID

A structural implementation demonstrates that it satisfies the required contract.

A ULABI profile MUST define which mechanism applies.

An implementation MUST NOT silently treat an unrelated object as conforming merely because it has methods with similar names.


---

23. Inheritance

ULABI MUST NOT require inheritance.

A language MAY use:

single inheritance;

multiple inheritance;

interface inheritance;

traits;

mixins;

composition;

delegation;

prototype inheritance;

no inheritance.


ULABI interoperability MUST occur at the contract boundary.


---

24. Inheritance Adaptation

If Language A has:

Child extends Parent

and Language B has no inheritance, the adapter MAY represent:

Child
 |
 +-- Parent Interface

through interface implementation or delegation.

Inheritance itself MUST NOT cross the boundary as a hidden requirement.


---

25. Composition

Composition is preferred where inheritance semantics cannot be safely represented.

Example:

Object A
   |
   +-- contains Object B

The adapter MAY expose B through a ULABI interface.


---

26. Delegation

An implementation MAY delegate an interface method:

ULABI Interface
      |
      v
Adapter
      |
      v
Internal Object

The delegation mechanism MUST preserve the interface contract.


---

27. Polymorphism

ULABI supports semantic polymorphism.

A consumer may depend on:

InterfaceID

rather than a concrete implementation type.

Example:

Storage
   |
   +-- FileStorage
   +-- MemoryStorage
   +-- NetworkStorage

The consumer only depends on the declared ULABI interface.


---

28. Substitutability

An implementation claiming to satisfy an interface MUST preserve the interface's semantic guarantees.

It MUST NOT merely provide methods with matching names.

For example:

read()

is insufficient evidence of interoperability.

The implementation must satisfy the complete:

Method Identity
Signature
Ownership
Lifetime
Error Contract
Effects
Capabilities
Execution Semantics
Compatibility

contract.


---

29. Properties

An object MAY expose properties.

A property MUST have explicit semantics.

Conceptually:

Property {
    property_id
    type
    readable
    writable
    ownership
    lifetime
    effects
    capabilities
}

A property MAY be implemented as:

field;

getter/setter;

method pair;

computed value;

remote query;

cached value.



---

30. Fields

Fields MAY be exposed only when the ULABI contract explicitly defines them as externally visible.

A field's memory offset MUST NOT automatically become its semantic identity.

Field identity is governed by the applicable type and data specifications.


---

31. Object Construction

Object creation MUST be explicitly defined.

Possible construction mechanisms include:

Constructor
Factory
Create Function
Deserialize
Clone
Open Resource
Acquire Handle

ULABI does not require one universal construction mechanism.


---

32. Constructor Contract

A constructor exposed through ULABI MUST define:

constructor identity;

parameters;

parameter types;

ownership;

capabilities;

effects;

errors;

resulting object type;

resulting object ownership;

lifetime.


Example:

create_database(config: DatabaseConfig)
    -> Result<DatabaseConnection, DatabaseError>


---

33. Object Destruction

Object destruction MUST NOT be assumed to be identical to language-level destructor behavior.

Possible lifecycle operations include:

Release
Close
Destroy
Dispose
Drop
Shutdown
Finalize

Only the semantic lifecycle contract is portable.


---

34. Resource Objects

An object representing an external resource SHOULD expose an explicit lifecycle.

Examples:

File
Socket
DatabaseConnection
GPUBuffer
Device
Process
SharedMemory

The contract SHOULD define:

Acquire
Use
Release

and failure behavior.


---

35. Idempotent Release

Where possible, resource-release operations SHOULD be idempotent.

For example:

release(resource)
release(resource)

may both succeed without causing corruption.

If release is not idempotent, that fact MUST be explicitly declared.


---

36. Ownership

Object ownership MUST be governed by the ULABI memory model.

Possible semantic states include:

Owned
Borrowed
Shared
ImmutableShared
Transferred
Moved
Released

The object model MUST NOT create an independent ownership system.


---

37. Borrowed Objects

A borrowed object MUST have an explicit lifetime.

A consumer MUST NOT retain a borrowed object beyond the permitted lifetime.

If a language cannot safely represent the lifetime, the adapter MUST:

1. copy the data where permitted;


2. acquire an owned handle;


3. create a safe proxy;


4. reject the operation.



Unsafe retention is prohibited.


---

38. Object Handles

Opaque handles SHOULD be used when direct object representation is unsafe.

Conceptually:

ULABI Handle {
    handle_type
    instance_identity
    ownership
    lifetime
    capabilities
}

A handle MUST NOT expose:

raw private pointers;

allocator addresses;

secret runtime metadata.



---

39. Handle Safety

A handle MUST be validated before use where required.

Invalid states include:

Unknown
Expired
Released
Revoked
WrongType
WrongCapability
WrongOwner
InvalidVersion

The implementation MUST fail safely.


---

40. Capability-Bearing Objects

An object MAY represent authority.

For example:

FileHandle
GPUContext
NetworkSocket
DeviceHandle

Possession of the object MAY grant access to a resource.

Therefore object handles MUST integrate with:

docs/security/capability-security.md

and:

docs/security/security-model.md

A handle MUST NOT grant capabilities beyond those declared by its contract.


---

41. Capability Attenuation

An implementation MAY derive a restricted object from a more powerful object.

Example:

FullStorage
    |
    v
ReadOnlyStorage

The derived object MUST NOT regain capabilities that were intentionally removed.


---

42. Revocation

If the applicable security profile supports revocation, object capabilities MAY be revoked.

After revocation:

Object
   |
   v
Capability Check
   |
   v
Denied

The object MUST NOT silently regain the revoked capability.


---

43. Object Equality

ULABI distinguishes:

Identity Equality

from:

Value Equality

Two object references may point to the same object identity.

Two separate objects may contain equal values.

The object contract MUST identify which equality semantics it supports.


---

44. Identity Equality

Identity equality means:

Object A identity == Object B identity

The implementation MUST NOT use pointer equality unless pointer identity is explicitly the ULABI identity mechanism.


---

45. Value Equality

Value equality compares semantic state.

A value-equality contract MUST specify:

compared fields;

normalization;

type compatibility;

floating-point behavior where applicable;

unknown-field behavior.



---

46. Hashing

If an object exposes hashing, the hash contract MUST specify whether hashing is based on:

identity;

semantic value;

stable serialized representation.


A process-local memory address MUST NOT be used as a portable semantic hash.


---

47. Object Mutability

Objects MUST explicitly declare whether externally observable state may change.

Possible states include:

Immutable
Mutable
AppendOnly
WriteOnce
CapabilityControlled
Unknown

A consumer MUST NOT assume mutability.


---

48. Thread Safety

Thread safety MUST be explicitly declared where relevant.

Possible object concurrency contracts include:

NotThreadSafe
ThreadCompatible
ThreadSafe
ExternallySynchronized
ActorIsolated
SingleOwner
Concurrent

These semantics belong to the object contract.

The object model MUST NOT assume a particular language threading model.


---

49. Reentrancy

A method contract SHOULD declare whether reentrant invocation is permitted.

Possible policies:

Reentrant
NonReentrant
ConditionallyReentrant
Serialized

A violation MUST produce defined behavior rather than undefined behavior.


---

50. Asynchronous Objects

Objects MAY expose asynchronous operations.

For example:

Future<Result<T,E>>

or:

AsyncObject

The asynchronous semantics are governed by the ULABI runtime/async specifications.

The object model only identifies the relationship between the asynchronous operation and the object.


---

51. Callback Objects

An object MAY receive callbacks.

A callback MUST have:

stable interface identity;

method identity;

lifetime;

ownership;

capability requirements;

execution semantics.


The callback MUST NOT outlive its declared lifetime.


---

52. Event Sources

An object MAY expose events or notifications.

An event contract MUST define:

EventIdentity
PayloadType
DeliverySemantics
Ordering
Lifetime
Cancellation
FailureBehavior

Events MUST NOT be treated as ordinary synchronous method calls unless the contract says so.


---

53. Actor-Like Objects

ULABI MAY represent actor-like systems as objects.

An actor-like object MAY have:

Identity
State
Message Interface
Mailbox
Lifecycle
Capabilities

ULABI does not require a specific actor runtime.

The message semantics MUST be explicit.


---

54. Proxy Objects

A proxy object represents another object.

Examples:

Local Object
    |
    v
Proxy
    |
    v
Remote Object

A proxy MUST NOT imply that remote execution is equivalent to local execution.

Latency, failure, security, and locality MUST remain explicit according to the distributed specifications.


---

55. Remote Object Identity

A remote object MAY have a stable identity.

The identity MUST distinguish:

Object Type
Object Instance
Authority / Endpoint Context

where required.

A remote reference MUST NOT be assumed to remain valid indefinitely.


---

56. Remote Object Failure

A remote object operation MAY fail because of:

network failure;

timeout;

object disappearance;

capability revocation;

version mismatch;

remote process failure;

transport failure.


These failures MUST NOT be silently converted into ordinary local object behavior.


---

57. Serialization Boundary

Object serialization is NOT defined by this specification.

Serialization is governed by:

docs/distributed/serialization.md

The object model only defines what object semantics MUST be preserved when serialization is explicitly supported.

Possible semantics:

CopyValue
Snapshot
Reference
Proxy
Opaque
Unsupported


---

58. Object Identity During Serialization

If an object is serialized as a value copy:

Object A
   |
   v
Serialized Value
   |
   v
Object B

A and B MUST NOT automatically be considered the same identity.

If identity preservation is required, the serialization/profile contract MUST explicitly define it.


---

59. Cyclic Object Graphs

Object graphs MAY contain cycles.

Example:

A -> B
^    |
|____|

A serialization mechanism supporting object graphs MUST explicitly define:

cycle detection;

identity preservation;

maximum graph depth;

resource limits;

failure behavior.


The object model does not prescribe a serialization algorithm.


---

60. Object Graph Limits

Implementations MUST apply resource limits when processing externally supplied object graphs.

Limits MAY include:

MaximumObjects
MaximumDepth
MaximumEdges
MaximumBytes
MaximumReferences
MaximumProcessingTime

Unbounded graph processing MUST NOT be an implicit requirement.


---

61. Reflection

Reflection is language-specific.

ULABI MUST NOT require universal reflection.

An object MAY expose explicit introspection metadata.

For example:

describe_type()
describe_interface()
describe_methods()
describe_properties()

If introspection is exposed, its security and information-disclosure properties MUST be defined.


---

62. Introspection

Introspection MAY reveal:

object type identity;

interface identities;

method identities;

property identities;

capability classes;

version;

supported profiles.


It MUST NOT reveal private implementation details unless explicitly authorized.


---

63. Generic Objects

A language MAY implement generic object types.

ULABI generic semantics are governed by the type-system specifications.

The object model MUST NOT require:

Java generics
C++ templates
Rust generics
Zamani generics
Sankofa generics

or any particular generic implementation.


---

64. Object Versioning

An object contract MUST have explicit version information where evolution is possible.

Versioning MUST distinguish:

Compatible Extension

from:

Breaking Change

The compatibility rules are governed by:

ULABI-VERSIONING.md

docs/compatibility/backwards-compatibility.md

docs/compatibility/forwards-compatibility.md

docs/compatibility/feature-negotiation.md



---

65. Interface Evolution

An interface MAY evolve by:

adding optional methods;

adding optional properties;

adding capabilities;

adding metadata;

introducing a new interface version.


Removing or changing required behavior MUST be treated according to the compatibility specification.


---

66. Unknown Methods

An implementation receiving an unknown method identity MUST NOT invoke an unrelated method merely because the names are similar.

Possible behavior:

UnsupportedMethod
UnknownMethod
VersionMismatch
CapabilityDenied

The behavior MUST be deterministic.


---

67. Security Boundary

Objects crossing a language boundary MUST be treated as untrusted unless their provenance has been validated according to the applicable security profile.

The adapter MUST validate:

object identity;

object type;

interface;

capabilities;

lifetime;

ownership;

version;

integrity where required.



---

68. No Arbitrary Pointer Exposure

ULABI object interoperability MUST NOT require raw pointer exposure.

An implementation MUST NOT expose private pointers merely to make two languages interoperable.

If a low-level pointer-based profile is introduced, it MUST define:

ownership;

lifetime;

alignment;

validity;

aliasing;

provenance;

revocation;

architecture requirements.



---

69. Capability Checks

Before executing a capability-sensitive object operation:

Object
  |
  v
Capability Check
  |
  +-- allowed --> Execute
  |
  +-- denied ---> Error

Capability checks MUST occur at the appropriate security boundary.

An object method MUST NOT silently escalate its caller's authority.


---

70. Effects

Object methods MAY declare effects such as:

Pure
ReadsState
WritesState
ReadsResource
WritesResource
Network
Filesystem
GPU
Device
Process
Time
Randomness
NonDeterministic

Effect semantics are governed by the broader ULABI ABI/runtime/security specifications.

The object contract consumes those declarations.


---

71. Failure Modes

Object operations MUST define relevant failures.

Possible failures include:

InvalidObject
ExpiredObject
ReleasedObject
InvalidMethod
InvalidArguments
CapabilityDenied
OwnershipViolation
LifetimeViolation
UnsupportedOperation
VersionMismatch
ResourceUnavailable
Timeout
Cancellation
RemoteFailure
InternalFailure

Implementations MUST NOT silently reinterpret one failure as another.


---

72. Recovery

Recovery behavior is governed by the reliability specifications.

An object model MAY expose safe lifecycle recovery such as:

Retry
Reconnect
Reopen
Reset
Reinitialize
Rollback

Recovery MUST be bounded and policy-controlled.

The object model MUST NOT authorize arbitrary self-modification.


---

73. Object State Corruption

If an object detects state corruption, the implementation MUST NOT continue operating as if the state were valid unless the applicable reliability contract explicitly permits recovery.

Possible behavior:

Detect
  |
Diagnose
  |
Isolate
  |
Recover
  |
Verify
  |
+----+----+
|         |
Healthy   Failed
|         |
Continue  Rollback / Escalate


---

74. Determinism

Where an object contract declares deterministic behavior, equivalent inputs and state MUST produce equivalent observable results according to the contract.

Implementation-specific:

allocation;

scheduling;

addresses;

internal caches;


MUST NOT become observable semantic differences unless explicitly declared.


---

75. Object Caching

An implementation MAY cache object state or method results.

Caching MUST NOT violate:

mutability;

freshness;

synchronization;

ownership;

security;

effect;

determinism.


A cache MUST NOT cause a consumer to observe behavior prohibited by the object contract.


---

76. Object Lifetime State Machine

A conforming implementation SHOULD model externally visible lifecycle states explicitly:

Uncreated
    |
    v
Created
    |
    v
Active
    |
    +------+
    |      |
    v      v
Closing  Failed
    |      |
    v      v
Released  Recovery
             |
             v
          Active / Failed

Invalid transitions MUST fail safely.


---

77. Object Contract Invariants

The following invariants are mandatory.

OMI-INV-001 — Stable Type Identity

A ULABI object type MUST have a stable semantic identity.

OMI-INV-002 — Stable Instance Identity

Where identity stability is promised, moving an object MUST NOT change its semantic identity.

OMI-INV-003 — No Layout Assumption

Consumers MUST NOT assume a foreign object has a compatible internal memory layout.

OMI-INV-004 — Explicit Ownership

Ownership MUST be explicit.

OMI-INV-005 — Explicit Lifetime

Foreign references MUST have defined lifetimes.

OMI-INV-006 — Capability Limitation

Objects MUST NOT grant undeclared capabilities.

OMI-INV-007 — Contract Validation

Method invocation MUST satisfy the applicable object/interface contract.

OMI-INV-008 — No Name-Only Dispatch

Source-level names MUST NOT be sufficient to establish semantic equivalence.

OMI-INV-009 — Safe Failure

Invalid object operations MUST fail according to the defined error contract.

OMI-INV-010 — Language Independence

No ULABI object contract may depend on one programming language.


---

78. Security Requirements

Implementations MUST:

1. validate object identity where required;


2. validate interface identity;


3. validate capabilities;


4. enforce ownership;


5. enforce lifetime;


6. reject unauthorized operations;


7. prevent raw pointer leakage;


8. prevent stale-handle use;


9. apply resource limits;


10. prevent object graph exhaustion;


11. validate externally supplied metadata;


12. avoid implicit privilege escalation.



Implementations SHOULD support:

capability revocation;

object provenance;

integrity verification;

audit events;

secure handles;

sandbox boundaries.



---

79. Performance Requirements

The object model MUST permit efficient implementations.

Possible implementations include:

Direct Call
Function Table
Interface Table
Generated Adapter
Inline Adapter
Handle Table
Proxy
Shared Memory
Remote Proxy

The specification MUST NOT require one implementation strategy.

Zero-copy operation MAY be used when the memory contract permits it.

Performance optimizations MUST NOT weaken semantic guarantees.


---

80. Zero-Copy Objects

Zero-copy interoperability MAY be used when:

ownership is safe;

lifetime is safe;

aliasing is safe;

representation is compatible;

mutation semantics are compatible;

capability boundaries are preserved.


Otherwise, the implementation MUST copy or use an opaque representation.


---

81. Cross-Language Object Adapter

A conforming language implementation SHOULD provide conceptual components:

ObjectAdapter
    |
    +-- ObjectTypeRegistry
    +-- ObjectIdentityManager
    +-- InterfaceRegistry
    +-- MethodRegistry
    +-- ObjectHandleManager
    +-- LifecycleManager
    +-- OwnershipAdapter
    +-- LifetimeManager
    +-- CapabilityAdapter
    +-- DispatchAdapter
    +-- PropertyAdapter
    +-- ErrorAdapter
    +-- CompatibilityValidator
    +-- IntrospectionAdapter
    +-- DiagnosticsAdapter

These are logical components.

They do not have to be separate source files.


---

82. Integration With Language Interoperability

docs/interoperability/language-interoperability.md defines the overall language interoperability architecture.

This document supplies its object-specific contract.

The relationship is:

Language Interoperability
          |
          v
    Object Model
          |
    +-----+-----+
    |     |     |
   FFI  Types  Memory

The object model MUST NOT redefine the complete language-interoperability architecture.


---

83. Integration With FFI

docs/interoperability/foreign-function-interface.md defines how language implementations bind foreign functions.

This document defines the semantic object behavior those functions MAY operate upon.

For example:

FFI
 |
 +-- create_object()
 +-- object_method()
 +-- release_object()

The FFI performs the binding.

The object model defines the object semantics.


---

84. Integration With Cross-Language Data

docs/interoperability/cross-language-data.md defines cross-language value mapping.

Objects SHOULD use:

Record
Variant
Option
Result
Handle
Capability

or another explicitly defined representation when crossing a language boundary.

Private object layouts MUST NOT be exposed merely for data conversion.


---

85. Integration With Name Mangling

docs/interoperability/name-mangling.md defines stable identity and linkage encoding.

Object type identities and method identities MUST use that identity system.

This document does not define another mangling scheme.


---

86. Integration With Symbol Resolution

docs/interoperability/symbol-resolution.md resolves externally visible identities.

The object model consumes resolved:

Object Type
Interface
Method
Property
Lifecycle Operation

identities.


---

87. Integration With Calling Convention

docs/abi/calling-convention.md defines how object methods are invoked at the ABI boundary.

The object model MUST NOT redefine:

register allocation;

stack layout;

argument passing;

return passing.


Object methods simply become ULABI callable contracts.


---

88. Integration With Memory Model

docs/abi/memory-model.md defines:

ownership;

lifetime;

borrowing;

sharing;

transfer;

release.


The object model consumes those semantics.

It MUST NOT create an independent memory ownership model.


---

89. Integration With Security

Object capabilities integrate with:

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md

Object identity MUST NOT be interpreted as authority unless the security contract explicitly defines it.


---

90. Integration With Distributed Objects

Distributed object behavior integrates with:

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md

Remote objects MUST preserve explicit locality semantics.

A remote object MUST NOT silently become indistinguishable from a local object when that difference affects:

latency;

failure;

consistency;

security;

resource ownership.



---

91. Integration With Compatibility

Object evolution integrates with:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Object implementations MUST use explicit compatibility mechanisms.


---

92. Integration With Reliability

Object failure and recovery integrate with:

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/self-healing.md

The object model MUST NOT authorize unrestricted autonomous modification.


---

93. Conformance Requirements

An implementation claiming ULABI Object Model conformance MUST demonstrate:

OMI-CON-001

Stable object type identity.

OMI-CON-002

Stable method identity.

OMI-CON-003

Correct interface validation.

OMI-CON-004

Correct ownership enforcement.

OMI-CON-005

Correct lifetime enforcement.

OMI-CON-006

Correct capability enforcement.

OMI-CON-007

No dependence on foreign object layout.

OMI-CON-008

Correct lifecycle behavior.

OMI-CON-009

Correct failure behavior.

OMI-CON-010

Correct compatibility behavior.

OMI-CON-011

Correct handling of invalid/stale handles.

OMI-CON-012

Correct object/interface version validation.

OMI-CON-013

Correct cross-language method invocation.

OMI-CON-014

Correct object identity behavior.

OMI-CON-015

Correct security boundary behavior.


---

94. Required Conformance Test Categories

The ULABI conformance suite MUST eventually test:

Object Identity
Object Type Identity
Interface Identity
Method Identity
Method Dispatch
Interface Dispatch
Object Construction
Object Destruction
Ownership
Borrowing
Lifetime
Handles
Capabilities
Revocation
Mutability
Equality
Hashing
Properties
Callbacks
Events
Async Objects
Proxy Objects
Remote Objects
Serialization Boundaries
Versioning
Compatibility
Invalid Handles
Invalid Methods
Capability Denial
Lifetime Violations
Resource Exhaustion
Object Graph Limits
Security Isolation


---

95. Required Negative Tests

The conformance suite MUST test failure cases, including:

WrongObjectType
UnknownObject
ReleasedObject
ExpiredObject
WrongInterface
UnknownMethod
SignatureMismatch
VersionMismatch
CapabilityDenied
OwnershipViolation
LifetimeViolation
InvalidHandle
UnauthorizedPropertyWrite
UnsupportedOperation
RemoteObjectUnavailable
ResourceExhaustion
MalformedMetadata
ObjectGraphOverflow

A conforming implementation MUST fail according to the applicable contract.


---

96. Reference Implementation Requirements

A reference implementation SHOULD demonstrate at least:

Object Registry
Interface Registry
Method Registry
Object Handles
Object Identity
Lifecycle Management
Ownership
Lifetime
Capabilities
Dynamic Dispatch
Interface Dispatch
Error Translation
Compatibility Validation

The reference implementation MUST remain independent of any particular programming language.

Multiple independent implementations SHOULD eventually exist.


---

97. Implementation Independence

A ULABI object implementation MAY be written in:

C;

C++;

Rust;

Go;

Java;

Python;

Swift;

Kotlin;

Fortran;

Ada;

or another language.


ULABI MUST NOT require any particular language.

Zamani and Sankofa MAY independently implement this specification.

Neither language is normative.


---

98. Minimal Core Rule

Object interoperability SHOULD remain outside the smallest possible ULABI Core where practical.

The Core requires only the object semantics necessary for stable interoperability.

Advanced object features SHOULD be delivered through profiles.

Possible profiles include:

Object Profile
Interface Profile
Resource Object Profile
Capability Object Profile
Distributed Object Profile
Actor Object Profile
Zero-Copy Object Profile
Reflection Profile
Real-Time Object Profile
Safety-Critical Object Profile


---

99. Failure-Oriented Object Design

Every externally visible object operation MUST define:

Preconditions
Operation
Postconditions
Failure Modes
Recovery
Resource Effects
Capability Effects
Lifetime Effects

The implementation MUST NOT rely on undocumented behavior.


---

100. Canonical Object Lifecycle

The conceptual lifecycle is:

Contract Discovery
       |
       v
Type Validation
       |
       v
Construction
       |
       v
Capability Validation
       |
       v
Active
       |
       +-------------------+
       |                   |
       v                   v
Method Invocation       Failure
       |                   |
       v                   v
State Update          Diagnosis
       |                   |
       v                   v
Continue              Recovery
                           |
                      +----+----+
                      |         |
                   Healthy    Failed
                      |         |
                      v         v
                   Active    Rollback /
                             Release /
                             Escalate


---

101. Normative Summary

A conforming ULABI object system MUST:

1. define stable object identities;


2. define stable object type identities;


3. define stable interface identities;


4. define stable method identities;


5. validate contracts;


6. preserve ownership;


7. preserve lifetime;


8. preserve capability boundaries;


9. avoid dependence on foreign object layouts;


10. support explicit lifecycle semantics;


11. define failure behavior;


12. preserve compatibility rules;


13. prevent unauthorized operations;


14. avoid raw pointer dependence;


15. support independent implementations;


16. remain language-neutral;


17. remain runtime-neutral;


18. remain architecture-neutral;


19. remain vendor-neutral.



The central rule is:

> ULABI standardizes object interoperability, not a universal object-oriented programming model.




---

102. Completion Criteria

This document is considered complete when:

all object semantics have normative requirements;

no language-specific object model is normative;

object identity is defined;

interface identity is defined;

method identity is defined;

lifecycle semantics are defined;

ownership/lifetime integration is explicit;

capability integration is explicit;

FFI integration is explicit;

cross-language data integration is explicit;

distributed integration is explicit;

compatibility integration is explicit;

security integration is explicit;

conformance requirements are defined;

negative tests are defined;

reference implementation requirements are defined.


No later document should need to rewrite this specification merely because another ULABI subsystem is implemented.

Later specifications should implement the contracts established here.

---

# 3. Why this document is deliberately different from the others

This avoids the repetition problem you are trying to eliminate.

| Concern | Owning document | `object-model.md` does |
|---|---|---|
| Universal primitive types | `docs/abi/data-types.md` | Consumes them |
| Calling mechanics | `docs/abi/calling-convention.md` | Uses them for methods |
| Memory ownership/lifetime | `docs/abi/memory-model.md` | Applies them to objects |
| Language interoperability | `language-interoperability.md` | Provides object-specific contract |
| FFI | `foreign-function-interface.md` | Defines object semantics for FFI |
| Data conversion | `cross-language-data.md` | Defines when objects become values/handles |
| Symbol identity | `name-mangling.md` | Uses object/method identity |
| Symbol lookup | `symbol-resolution.md` | Consumes resolved object symbols |
| Serialization | `distributed/serialization.md` | Defines object semantics that serialization must preserve |
| Remote calls | `distributed/remote-calls.md` | Defines remote-object semantics |
| Security | `security/*` | Applies capability/security rules to objects |
| Compatibility | `compatibility/*` | Applies evolution rules to objects |
| Reliability | `reliability/*` | Applies failure/recovery rules to objects |

This is consistent with the repository's existing separation of responsibilities rather than creating another competing ABI model. The current repository already has separate core ABI, calling convention, data types, memory model, distributed serialization, and compatibility specifications.    

---

# 4. What should be done under which files

The correct implementation order is **not** to start writing all object-related code simultaneously.

The independent contract comes first.

## Phase 1 — independent specification

### 1. `docs/interoperability/object-model.md`

**This is the file you should complete first.**

It owns:

- object identity;
- object type identity;
- instance identity;
- interfaces;
- methods;
- properties;
- lifecycle;
- handles;
- object capabilities;
- polymorphism;
- dispatch semantics;
- object equality;
- object mutability;
- object failure;
- object conformance.

**Integration already specified:** FFI, types, memory, security, compatibility, distributed execution, name mangling, symbol resolution, reliability.

Once this file is completed, it should not need rewriting merely because implementation modules are created.

---

# 5. Files that should integrate with it afterward

## `docs/interoperability/language-interoperability.md`

Owns the **overall language-to-language architecture**.

It should not define object internals.

Integration:

```text
Language Interoperability
        |
        +--> Object Model
        +--> FFI
        +--> Cross-Language Data
        +--> Symbol Identity

The repository's current language-interoperability design already explicitly places the object model as a separate component.


---

docs/interoperability/foreign-function-interface.md

Owns:

importing;

exporting;

binding;

function invocation;

dynamic loading;

ABI binding.


Object model owns:

what an object is;

what an object method means;

object lifecycle;

object identity.


The existing FFI design already separates these responsibilities.


---

docs/interoperability/cross-language-data.md

Owns:

type mapping;

conversion;

lossless/lossy conversion;

records;

variants;

collections;

handles as data boundaries.


Object model consumes those mechanisms.

The existing cross-language-data document already explicitly says it should not redefine the universal types, general ABI, memory model, serialization, remote calls, or compatibility negotiation.


---

docs/interoperability/name-mangling.md

Owns:

canonical identity;

namespace;

entity identity;

operation identity;

linkage encoding;

symbol mangling.


Object model consumes it for:

ObjectTypeID
InterfaceID
MethodID
PropertyID
LifecycleOperationID

The repository's name-mangling design already explicitly separates source names from ULABI semantic identity.


---

docs/interoperability/symbol-resolution.md

Owns:

Identity → implementation

Object model should never implement its own symbol-resolution system.


---

6. Required schema files

After the normative document, create the machine-readable object metadata schemas.

Recommended:

schemas/
├── object-contract.schema.json
├── object-type.schema.json
├── object-instance.schema.json
├── interface-contract.schema.json
├── method-contract.schema.json
├── property-contract.schema.json
├── lifecycle-contract.schema.json
├── object-handle.schema.json
├── object-capability.schema.json
├── object-version.schema.json
└── object-error.schema.json

object-contract.schema.json

Defines:

object_type_id
version
interfaces
methods
properties
lifecycle
ownership
capabilities
effects
execution
compatibility

object-type.schema.json

Defines stable object type metadata.

object-instance.schema.json

Defines runtime instance metadata without exposing implementation-specific memory addresses.

interface-contract.schema.json

Defines interface identity and required operations.

method-contract.schema.json

Defines method identity and semantic signature.

property-contract.schema.json

Defines property semantics.

lifecycle-contract.schema.json

Defines construction/release/close/reset behavior.

object-handle.schema.json

Defines opaque handles.

object-capability.schema.json

Defines authority associated with an object.


---

7. Required examples

Create:

examples/
└── object-model/
    ├── basic-object/
    ├── interface-object/
    ├── opaque-handle/
    ├── resource-object/
    ├── capability-object/
    ├── callback-object/
    ├── async-object/
    ├── proxy-object/
    ├── remote-object/
    └── cross-language-object/

Each example should demonstrate the ULABI contract, not a particular language's object system.

For example:

examples/object-model/interface-object/
├── interface.ulabi
├── object.ulabi
└── README.md


---

8. Required conformance tests

Create:

tests/object-model/
├── identity/
├── interfaces/
├── methods/
├── lifecycle/
├── ownership/
├── lifetime/
├── handles/
├── capabilities/
├── dispatch/
├── properties/
├── equality/
├── mutation/
├── callbacks/
├── async/
├── proxies/
├── remote/
├── compatibility/
├── security/
└── failures/

Important tests include:

object_identity_stable
object_type_identity_stable
method_identity_stable
interface_identity_stable

wrong_object_type_rejected
wrong_method_rejected
released_handle_rejected
expired_handle_rejected

ownership_violation_rejected
lifetime_violation_rejected
capability_violation_rejected

interface_version_mismatch_rejected
method_signature_mismatch_rejected

immutable_object_cannot_be_mutated
read_only_interface_cannot_write

object_move_preserves_identity


---

9. Required conformance modules

Create a conceptual conformance layer:

conformance/object_model/
├── object_identity
├── object_types
├── interfaces
├── methods
├── dispatch
├── lifecycle
├── ownership
├── lifetime
├── handles
├── capabilities
├── properties
├── equality
├── mutation
├── callbacks
├── async
├── proxies
├── remote_objects
├── compatibility
├── security
└── failures

These are logical modules and can later be implemented in whatever language the conformance framework uses.


---

10. Required implementation modules

The object implementation should eventually contain these logical modules:

object/
├── object_contract
├── object_type_registry
├── object_identity
├── object_instance
├── interface_registry
├── interface_contract
├── method_registry
├── method_contract
├── method_dispatch
├── property_contract
├── lifecycle_manager
├── constructor_adapter
├── destructor_adapter
├── resource_lifecycle
├── object_handle
├── handle_registry
├── ownership_adapter
├── lifetime_manager
├── capability_adapter
├── capability_check
├── capability_revocation
├── object_proxy
├── remote_object
├── callback_adapter
├── event_adapter
├── async_object
├── object_equality
├── object_hashing
├── object_introspection
├── compatibility_validator
├── object_error_adapter
├── object_security
├── object_diagnostics
└── object_validator

These should initially be treated as logical modules, not mandatory filenames.

That keeps ULABI language-neutral.


---

11. Required integration modules

The object implementation should then connect to the wider ULABI implementation through:

abi/
├── core_abi
├── calling_convention
├── data_types
└── memory_model

interoperability/
├── language_adapter
├── ffi
├── cross_language_data
├── name_mangling
└── symbol_resolution

security/
├── security_model
├── capabilities
└── sandbox

compatibility/
├── versioning
├── feature_negotiation
└── capability_discovery

distributed/
├── serialization
├── remote_calls
└── distributed_abi

runtime/
├── process
├── threading
├── async
└── resource_management

The object layer depends on these contracts; it should not duplicate them.


---

12. Final dependency graph

The clean architecture becomes:

ULABI-DESIGN
                             |
                       ULABI-SPEC
                             |
                    +--------+--------+
                    |                 |
                 Core ABI        Type System
                    |                 |
                    +--------+--------+
                             |
                    Memory Boundary
                             |
                             v
                LANGUAGE INTEROPERABILITY
                             |
             +---------------+---------------+
             |               |               |
            FFI       CROSS-LANGUAGE DATA   OBJECT MODEL
             |               |               |
             +---------------+---------------+
                             |
                 +-----------+-----------+
                 |                       |
           NAME MANGING           SYMBOL RESOLUTION
                 |                       |
                 +-----------+-----------+
                             |
                    LANGUAGE ADAPTERS
                             |
             +---------------+---------------+
             |               |               |
          Language A     Language B      Language C

And separately:

OBJECT MODEL
                         |
        +----------------+----------------+
        |                |                |
     Security       Compatibility     Reliability
        |                |                |
   Capabilities      Versioning       Recovery
        |                |                |
        +----------------+----------------+
                         |
                  Distributed Layer
                         |
                 Remote Object Model

The key architectural decision

Do not make ULABI's object model equivalent to a class system.

The universal abstraction should be:

> Object = stable identity + contract + state boundary + behavior + lifecycle + authority.



A class, struct, trait, protocol, actor, closure, handle, service, device object, or remote proxy can independently implement that contract.

That gives ULABI a much stronger universal model than simply standardizing something resembling C++/Java/Rust objects, while preserving the core principle already established in the repository: ULABI is the interoperability contract, not a programming language or runtime.