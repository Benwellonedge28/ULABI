I inspected the actual Benwellonedge28/ULABI repository before drafting this. The current ULABI-DESIGN.md establishes ULABI as language/runtime/platform/architecture/vendor neutral, with explicit ownership and lifetime boundaries, safe memory interoperability, zero-copy where safe, capability security, process/distributed modes, deterministic behavior, profiles, and conformance.

I also checked the existing memory documentation. docs/abi/memory-model.md, docs/memory/allocation.md, docs/memory/lifetimes.md, and docs/memory/memory-safety.md already cover or are intended to cover adjacent concerns; docs/memory/shared-memory.md is currently only a placeholder.

Therefore, docs/memory/shared-memory.md should not redefine ownership, allocation, lifetime, or general memory safety. It should define the specific contract for multiple independent participants accessing the same memory resource.

Below is the complete document I recommend copying into:

docs/memory/shared-memory.md


---

ULABI Shared Memory Specification

Universal Language Application Binary Interface

Project: ULABI
Full Name: Universal Language Application Binary Interface
Repository: Benwellonedge28/ULABI
Document: docs/memory/shared-memory.md
Specification Area: Memory / Shared Memory
Status: Architecture / Design Specification
Specification Version: 0.2.0-draft
License: Apache-2.0


---

1. Purpose

This document defines the ULABI Shared Memory contract.

The purpose of shared memory in ULABI is to allow two or more independently implemented components to access a common memory resource while preserving:

memory safety;

ownership correctness;

lifetime correctness;

authorization;

visibility guarantees;

synchronization semantics;

data-race semantics;

isolation boundaries;

deterministic behavior where required;

compatibility across languages and runtimes;

portability across operating systems and architectures.


Shared memory is an optimization and interoperability mechanism, not a requirement that all ULABI implementations expose raw memory.

ULABI shared memory may be implemented using:

operating-system shared memory;

memory-mapped files;

shared pages;

shared buffers;

ring buffers;

lock-free structures;

synchronized regions;

hardware-supported shared memory;

device memory;

accelerator memory;

language-runtime-managed shared regions;

capability-controlled memory mappings;

another implementation mechanism.


The implementation mechanism is not itself part of the universal contract.

The observable semantics are.


---

2. Relationship to ULABI

ULABI-DESIGN.md establishes ULABI as a language-neutral interoperability contract rather than a universal memory-management system. It explicitly includes safe memory interoperability, zero-copy interoperability where safe, ownership boundaries, capability security, process isolation, distributed interoperability, and deterministic behavior.

The ULABI memory model already defines the general concept of a memory resource, including:

identity;

ownership;

access rights;

lifetime;

locality;

mutability;

sharing policy;

capabilities.


Therefore this document specializes those concepts for concurrent or multi-party access.

The relationship is:

ULABI
                           |
                     Memory Contract
                           |
                    Memory Model
                           |
                 +---------+---------+
                 |                   |
             Ownership            Lifetime
                 |                   |
                 +---------+---------+
                           |
                     Shared Memory
                           |
             +-------------+-------------+
             |             |             |
          Access       Synchronization  Visibility
          Rights           Rules         Rules
             |             |             |
             +-------------+-------------+
                           |
                      Memory Safety
                           |
                      Implementations

This document does not redefine:

general ownership;

general lifetime;

allocation;

virtual memory;

general memory safety;

serialization;

capability authorization.


Those remain defined by their respective specifications.


---

3. Normative Language

The following terms are normative:

MUST

MUST NOT

REQUIRED

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY


A conforming implementation MUST satisfy all applicable MUST and MUST NOT requirements.


---

4. Fundamental Principle

> Shared memory MUST NEVER imply unrestricted shared authority.



The existence of a common memory region does not by itself grant every participant permission to:

read;

write;

resize;

remap;

transfer;

release;

revoke;

synchronize;

reinterpret;

destroy


that resource.

Conceptually:

Shared Memory
                       |
          +------------+------------+
          |            |            |
       Process A    Process B    Process C
          |            |            |
       READ/WRITE   READ ONLY     READ ONLY

Each participant MUST have explicitly defined rights.


---

5. Shared Memory Definition

A ULABI shared-memory resource is a memory resource that may be accessed by more than one independently executing participant under an explicit sharing contract.

Conceptually:

SharedMemoryResource {
    resource_id
    type
    size
    alignment
    memory_domain
    participants
    access_rights
    ownership_model
    lifetime
    synchronization_model
    visibility_model
    consistency_model
    capability_requirements
}

The physical representation is implementation-defined.


---

6. Shared Memory Is a Resource

A shared-memory region MUST be treated as an explicit ULABI resource.

It MUST NOT be treated merely as:

address

or:

pointer

A raw address is insufficient to establish:

ownership;

authorization;

lifetime;

size;

domain;

sharing state;

synchronization;

validity.



---

7. Resource Identity

Every shared-memory resource MUST have a stable semantic identity within its applicable scope.

Identity MUST NOT depend solely upon:

virtual address;

physical address;

process-local pointer;

allocator address;

CPU-specific address;

temporary mapping address.


A mapping may change while the underlying shared-memory resource remains the same.

Conceptually:

Resource ID
    |
    +---- Process A mapping: 0x100000
    |
    +---- Process B mapping: 0x7F000000
    |
    +---- Process C mapping: 0x40000000

The mappings may differ while the resource identity remains stable.


---

8. Sharing Scope

A shared-memory resource MUST declare or otherwise establish its sharing scope.

Possible scopes include:

ThreadLocal
ProcessLocal
HostLocal
ContainerLocal
VirtualMachineLocal
DeviceLocal
MultiProcess
CrossHost
Distributed

Not every implementation must support every scope.

A resource MUST NOT silently change from one scope to another.

For example:

ProcessLocal

MUST NOT silently become:

NetworkAccessible

merely because an implementation chooses to virtualize it.


---

9. Local Shared Memory

Local shared memory exists within a defined host or execution environment.

Examples include:

threads sharing a heap;

processes sharing mapped pages;

operating-system shared-memory segments;

shared memory between language runtimes.


The implementation MUST define:

participating execution contexts;

memory mapping semantics;

synchronization semantics;

lifetime;

permissions.



---

10. Cross-Process Shared Memory

Cross-process shared memory requires explicit process-boundary semantics.

A process MUST NOT gain access merely because it knows:

an address;

a resource name;

a resource identifier.


Authorization MUST be independently established.

Conceptually:

Process A
    |
    | capability / authorized mapping
    v
Shared Memory
    ^
    | capability / authorized mapping
    |
Process B


---

11. Cross-Host Shared Memory

ULABI MUST NOT pretend that network communication and shared memory have identical semantics.

True physical shared memory generally does not exist between independent hosts in the ordinary sense.

A system that provides a distributed memory abstraction MUST explicitly declare that it is a distributed memory profile rather than silently treating remote communication as local memory.

Differences such as:

latency;

failure;

consistency;

partitioning;

ordering;

availability;


MUST remain observable where they affect correctness.


---

12. Ownership Models

Shared memory MAY use different ownership models.

Possible models include:

12.1 Single Owner

One participant owns the resource while others receive controlled access.

12.2 Shared Ownership

Multiple participants collectively hold ownership authority under an explicitly defined protocol.

12.3 External Owner

A separate resource manager owns the region and grants access to participants.

12.4 Immutable Shared Ownership

Multiple participants may read the resource while no participant has mutation authority.

12.5 Capability-Based Ownership

Access is represented by explicit capabilities rather than language-level ownership.

ULABI does not require one universal ownership model.

The selected model MUST be explicit.


---

13. Access Rights

Each participant MUST have defined access rights.

At minimum, the model SHOULD distinguish:

None
Read
Write
ReadWrite
Map
Unmap
Resize
Transfer
Release
Share
Revoke
Synchronize

Not every implementation must expose every right.

Rights MUST NOT be inferred merely from membership in a shared-memory group.


---

14. Read-Only Sharing

Read-only sharing is the preferred shared-memory mechanism when mutation is unnecessary.

Example:

Immutable Region
              /      |      \
             /       |       \
          Reader   Reader   Reader

Multiple participants MAY read simultaneously without write synchronization.

Once established as immutable, the implementation MUST prevent unauthorized mutation.


---

15. Mutable Shared Memory

Mutable shared memory requires an explicit synchronization model.

The following is insufficient:

Shared Buffer
    |
Everyone may write

A mutable shared resource MUST specify how concurrent writes are coordinated.

Possible mechanisms include:

mutexes;

read/write locks;

atomic operations;

transactional memory;

ownership protocols;

message passing;

lock-free algorithms;

wait-free algorithms;

versioning;

sequence counters;

another formally defined synchronization mechanism.



---

16. Synchronization Contract

Every mutable shared-memory resource MUST define its synchronization semantics.

The contract SHOULD specify:

synchronization primitives;

lock ownership;

lock scope;

atomicity;

ordering;

visibility;

fairness where applicable;

blocking behavior;

timeout behavior;

cancellation behavior;

failure behavior.



---

17. Synchronization Is Not Ownership

Synchronization and ownership are distinct.

For example:

Lock acquired

does not necessarily mean:

Resource ownership transferred

Likewise:

Ownership granted

does not automatically establish:

Memory visibility

Implementations MUST NOT silently conflate these concepts.


---

18. Atomicity

An operation declared atomic MUST appear indivisible according to the applicable memory-consistency contract.

If an operation is not atomic, the implementation MUST NOT claim atomic semantics.

Atomicity MUST be defined relative to:

operation;

participants;

memory location or region;

ordering model.



---

19. Alignment of Atomic Operations

Atomic operations MUST satisfy the alignment requirements of the target representation and execution environment.

An implementation MUST NOT expose an operation as atomic if its underlying representation cannot satisfy the required atomicity guarantees.


---

20. Memory Ordering

Where shared-memory mutation is supported, the contract MUST define the relevant memory-ordering semantics.

Possible ordering classes include:

Relaxed
Acquire
Release
AcquireRelease
SequentiallyConsistent

The actual set may be profile-specific.

A language or runtime MUST NOT assume stronger ordering than the ULABI contract provides.


---

21. Visibility

A write becoming locally complete does not automatically establish that another participant can immediately observe it.

The contract MUST define visibility semantics.

Conceptually:

Writer
  |
  | write
  v
Memory
  |
  | synchronization / visibility rule
  v
Reader

A conforming implementation MUST preserve the declared visibility guarantees.


---

22. Data Race Semantics

A data race occurs when conflicting concurrent accesses occur without the synchronization required by the applicable contract.

ULABI implementations MUST explicitly define the behavior of data races.

Possible policies include:

1. data races prohibited;


2. data races detected;


3. data races produce defined failure;


4. data races permitted only under an explicitly defined weak-memory profile.



A conforming safety profile SHOULD prohibit undefined behavior caused by cross-boundary data races.


---

23. No Undefined Cross-Boundary Memory Corruption

A participant MUST NOT be allowed to corrupt another participant's unrelated ULABI resources merely through invalid shared-memory access.

Where isolation is technically possible, implementations SHOULD enforce:

Shared Region
      |
      X
Unrelated Memory

Cross-boundary corruption MUST be prevented or detected.


---

24. Bounds

Every shared-memory region MUST have authoritative bounds.

Access MUST satisfy:

0 <= offset
AND
size >= 0
AND
offset + size <= region_size

The arithmetic MUST be overflow-checked.

This document relies on docs/memory/memory-safety.md for the general memory-safety requirements.


---

25. Lifetime

A shared-memory region MUST have an explicit lifetime.

Possible models include:

CreatorLifetime
OwnerLifetime
SessionLifetime
ProcessLifetime
ExplicitLifetime
ReferenceLifetime
PersistentLifetime

The selected model MUST be declared.

A participant MUST NOT assume that another participant's continued existence guarantees the resource remains valid.


---

26. Lifetime Independence

A participant MAY need the resource to survive the termination of another participant.

If so, the shared-memory contract MUST define the corresponding lifetime mechanism.

For example:

Process A creates region
        |
        v
Process A exits
        |
        v
Region survives
        |
        v
Process B continues

This requires an explicit lifetime policy.

It MUST NOT be accidental.


---

27. Participant Termination

The contract MUST define what happens when a participant:

exits normally;

crashes;

is killed;

loses authorization;

disconnects;

becomes unresponsive.


Possible policies include:

resource remains available;

resource becomes read-only;

resource is revoked;

resource is destroyed;

recovery protocol starts;

ownership is transferred;

access is quarantined.


The policy MUST be deterministic enough for implementations to conform.


---

28. Revocation

Shared-memory access MAY be revoked.

Revocation MUST invalidate the applicable authority.

Conceptually:

Capability
    |
  revoke
    |
    v
No Access

After revocation, a participant MUST NOT continue accessing the resource merely because it retained:

a pointer;

a mapping;

a handle;

a cached capability;

a language object.



---

29. Mapping

A shared-memory resource MAY be mapped into multiple address spaces.

Each mapping MUST be treated as a distinct implementation-level mapping of the same semantic resource.

Conceptually:

Shared Resource
                   |
        +----------+----------+
        |          |          |
      Map A      Map B      Map C
        |          |          |
     Process A  Process B  Process C

A mapping MUST NOT change the resource's ownership or authorization semantics unless explicitly specified.


---

30. Mapping Permissions

Mappings MUST have explicit permissions.

Typical permissions include:

Read
Write
Execute
ReadWrite

Executable shared memory SHOULD be disabled by default unless explicitly required by a profile.

An implementation MUST NOT silently grant stronger permissions than requested.


---

31. Remapping

A mapping MAY move to another virtual address.

This MUST NOT change the semantic identity of the shared resource.

Any portable reference mechanism MUST therefore avoid depending solely upon mapping addresses.


---

32. Resizing

Resizing a shared-memory region is a high-risk operation.

A region MUST NOT be resized while another participant may hold an invalidated view unless the contract explicitly defines safe resize semantics.

Possible policies include:

Resize prohibited
Resize requires exclusive access
Resize requires quiescence
Resize creates a new generation
Resize invalidates old views
Resize is transactional

The selected policy MUST be explicit.


---

33. Stale Views

A view derived from shared memory MUST NOT remain valid after the underlying region changes in a way that invalidates the view.

Conceptually:

Region Generation 7
      |
     View
      |
   Resize
      |
Region Generation 8

The old view MUST either:

remain valid under the contract; or

be invalidated and rejected.


It MUST NOT silently reference incompatible memory.


---

34. Generation Tracking

Implementations SHOULD use generation numbers or equivalent mechanisms when shared resources can be invalidated and recreated.

Example:

resource_id = 42
generation = 7

After replacement:

resource_id = 42
generation = 8

A stale generation MUST NOT gain access to the new resource.


---

35. Zero-Copy

Shared memory MAY be used to provide zero-copy interoperability.

However:

> Zero-copy is an optimization, not a semantic requirement.



A zero-copy implementation MUST preserve the same:

type semantics;

ownership;

lifetime;

bounds;

permissions;

synchronization;

visibility;

security guarantees


as a copied implementation.


---

36. Zero-Copy Fallback

If zero-copy cannot be performed safely, the implementation MAY fall back to copying.

Example:

Safe zero-copy possible?
        |
      +---+
      |   |
     YES  NO
      |    |
  shared   copy
  memory   data

The fallback MUST preserve the semantic contract.

A system MUST NOT disable safety merely to preserve zero-copy performance.


---

37. Copy and Shared Memory

A participant MAY copy data out of shared memory where permitted.

A copied value becomes an independent resource unless the contract explicitly defines another relationship.

Shared Memory
      |
     copy
      |
      v
Independent Value

Modification of the copy MUST NOT implicitly modify the original shared region.


---

38. Immutable Snapshots

Implementations SHOULD support immutable snapshots where practical.

A snapshot can provide:

Shared Mutable State
        |
      snapshot
        |
        v
Immutable View

Snapshots can reduce synchronization requirements and simplify cross-language interoperability.


---

39. Versioned Shared Memory

A shared-memory region MAY use versioning.

Example:

Header {
    version
    generation
    size
    flags
}

Consumers MUST validate the version before interpreting the region.

Unknown versions MUST produce defined behavior.


---

40. Schema Validation

Structured shared-memory regions SHOULD include sufficient schema identity to determine whether the consumer understands the layout.

Schema identity MAY include:

type identifier;

schema identifier;

version;

representation version;

feature flags;

compatibility metadata.


A consumer MUST NOT interpret an incompatible layout as though it were compatible.


---

41. Type Safety

Shared memory MUST NOT bypass ULABI type safety.

For example:

Bytes

MUST NOT automatically become:

Record<T>

merely because a participant casts the region.

The interpretation MUST be authorized by the shared-memory contract.


---

42. Layout Stability

A shared-memory layout intended for direct binary access MUST define:

size;

alignment;

field offsets;

field representation;

endianness;

padding;

version;

compatibility rules.


Native language layout MUST NOT automatically become ULABI shared-memory layout.


---

43. Padding

Uninitialized padding MUST NOT become shared semantic data.

Padding MUST NOT be relied upon for:

synchronization;

identity;

hashing;

cryptographic signatures;

semantic comparison.


This follows the memory-safety and canonical-representation requirements.


---

44. Cache Coherence

Implementations operating on hardware with caches MUST preserve the declared ULABI visibility semantics.

Where explicit cache management is required, the applicable hardware/platform profile MUST define it.

Portable ULABI contracts SHOULD avoid exposing cache-maintenance instructions as universal semantics.


---

45. NUMA

NUMA topology MAY affect performance but MUST NOT silently alter semantic correctness.

A shared-memory contract MUST NOT assume:

same NUMA node

unless explicitly required by a specialized profile.


---

46. Device and Accelerator Shared Memory

Shared memory MAY exist between:

CPU and GPU;

CPU and NPU;

CPU and FPGA;

accelerator and accelerator;

host and device.


Such resources MUST explicitly define:

memory domain;

address-space semantics;

visibility;

synchronization;

ownership;

mapping;

transfer;

coherence;

lifetime.


Device memory MUST NOT automatically be treated as ordinary host memory.


---

47. Persistent Shared Memory

Persistent memory MAY survive process termination.

If supported, the contract MUST define:

persistence boundary;

initialization state;

recovery state;

consistency;

versioning;

crash behavior;

schema compatibility.


Persistence MUST NOT be inferred merely because a memory mapping exists.


---

48. Security

Shared memory MUST be subject to the ULABI security model.

Access MUST be authorized.

The shared-memory mechanism MUST NOT bypass:

capability restrictions;

process isolation;

sandboxing;

authentication;

authorization;

resource quotas.


A participant with read access MUST NOT automatically obtain write access.


---

49. Capability-Based Shared Memory

A capability MAY represent authority to access a shared-memory region.

A capability SHOULD encode or reference sufficient information to establish:

resource identity;

permitted operations;

scope;

expiration;

revocation;

participant identity where applicable.


Possession of a resource identifier alone MUST NOT constitute authorization.


---

50. Capability Attenuation

A participant SHOULD be able to derive a weaker capability from a stronger capability.

Example:

ReadWrite
    |
    v
ReadOnly

The reverse MUST NOT be possible without an explicit authorization event.


---

51. Capability Revocation

Revocation MUST take precedence over previously granted ordinary access.

A revoked participant MUST NOT regain access merely by:

retaining an old pointer;

retaining an old mapping;

replaying an old descriptor;

recreating an equivalent handle.



---

52. Sandboxing

Sandboxed implementations MUST treat shared memory as an explicitly granted capability.

A sandbox MUST NOT automatically receive access to:

arbitrary host memory;

arbitrary processes;

arbitrary device memory;

arbitrary shared-memory segments.



---

53. Resource Exhaustion

Shared-memory creation and mapping MUST be subject to resource controls where applicable.

Implementations SHOULD define limits for:

total shared memory;

per-resource size;

number of mappings;

number of participants;

synchronization objects;

outstanding views;

pinned memory;

device memory.


Failure due to quota exhaustion MUST be explicit.


---

54. Deadlocks

Synchronization mechanisms MUST document deadlock behavior.

Where locks are used, implementations SHOULD provide mechanisms for:

lock ordering;

timeout;

cancellation;

deadlock detection;

recovery.


A conforming implementation MUST NOT claim deadlock-free behavior unless the applicable profile establishes it.


---

55. Lock Ownership

A lock-based shared-memory contract MUST define:

who may acquire the lock;

whether locks are recursive;

whether locks are reentrant;

what happens if the owner terminates;

whether ownership can be recovered;

timeout semantics.



---

56. Crash Recovery

If a participant holding synchronization state crashes, the shared-memory contract MUST define the resulting state.

Possible states include:

Locked
Recoverable
Abandoned
Invalid
Quarantined

The implementation MUST NOT leave the state semantically ambiguous.


---

57. Lock-Free Shared Memory

ULABI MAY support lock-free data structures.

Lock-free behavior MUST be explicitly declared.

A lock-free structure MUST still define:

atomic operations;

memory ordering;

ownership;

lifetime;

reclamation;

ABA protection where relevant;

failure behavior.


Lock-free MUST NOT mean unsafe.


---

58. Memory Reclamation

Shared memory creates additional reclamation challenges.

A resource MUST NOT be reclaimed while a valid participant can still access it.

Possible mechanisms include:

reference counting;

epochs;

hazard pointers;

read-copy-update;

generation tracking;

ownership transfer;

explicit quiescence;

garbage collection;

another formally defined reclamation mechanism.


The implementation strategy is implementation-defined.

The safety property is normative.


---

59. ABA and Generation Safety

Where an identifier or address can be reused, implementations SHOULD protect against ABA-style confusion.

Example:

A -> B -> A

An observer MUST NOT incorrectly conclude that no relevant state transition occurred merely because an identifier returned to the same value.

Generation counters or equivalent mechanisms SHOULD be used where required.


---

60. Quiescence

A shared-memory operation MAY require all participants to reach a quiescent state.

A quiescence protocol MUST define:

who participates;

how participation is detected;

timeout;

failure;

cancellation;

completion.



---

61. Synchronization Failure

If synchronization cannot be established, the operation MUST fail safely.

It MUST NOT proceed as though synchronization succeeded.

Examples:

lock acquisition failure
memory fence failure
mapping failure
participant unavailable
authorization failure
resource invalidation

must result in defined behavior.


---

62. Partial Failure

Shared memory across multiple processes creates partial-failure scenarios.

The contract MUST define what happens when:

A healthy
B crashed
C disconnected

The state of the shared resource MUST remain governed by the declared recovery model.


---

63. Isolation

A shared-memory resource MUST be isolated from unrelated memory.

An invalid operation on shared region R1 MUST NOT silently modify unrelated region R2.

Where hardware or operating-system protection permits, implementations SHOULD use it.


---

64. Memory Corruption Detection

Implementations MAY use:

guard pages;

canaries;

checksums;

memory tagging;

hardware protection;

bounds metadata;

generation identifiers;

sanitizers.


These are implementation mechanisms.

The conformance requirement is that invalid shared-memory behavior MUST NOT become valid semantic behavior.


---

65. Determinism

Where deterministic execution is required by a ULABI profile, shared-memory behavior MUST be sufficiently constrained to reproduce the declared observable results.

Implementations SHOULD avoid relying on unspecified:

scheduling;

race ordering;

visibility timing;

lock acquisition ordering.



---

66. Real-Time Shared Memory

A real-time profile MAY impose:

bounded lock acquisition;

bounded memory access;

bounded reclamation;

deterministic priority behavior;

priority-inversion controls;

non-blocking operations.


Real-time guarantees MUST be explicitly declared.


---

67. Async Shared Memory

Asynchronous operations accessing shared memory MUST preserve:

lifetime;

ownership;

authorization;

synchronization;

cancellation;

resource validity.


A buffer MUST NOT be released merely because the initiating function returned if an asynchronous operation still depends upon it.


---

68. Cancellation

If an asynchronous shared-memory operation is cancelled, the contract MUST define:

whether the operation stopped;

whether memory remains accessible;

whether partial writes occurred;

whether rollback occurs;

whether the resource remains valid.


Cancellation MUST NOT silently create a lifetime violation.


---

69. Transactions

ULABI MAY support transactional shared-memory profiles.

A transaction SHOULD define:

Begin
Read
Write
Validate
Commit
Abort

Transaction semantics MUST specify visibility and isolation.


---

70. Snapshot Isolation

A shared-memory profile MAY provide snapshot isolation.

If supported, readers MUST observe a defined snapshot rather than an unspecified mixture of versions.


---

71. Consistency Models

Shared-memory implementations MAY expose different consistency models.

Examples include:

Strong
Sequentially Consistent
Causal
Eventual
Relaxed
Application Defined

The selected model MUST be explicitly declared.

A consumer MUST NOT assume stronger consistency than the contract provides.


---

72. Memory Ordering vs Consistency

Memory ordering and data consistency are related but distinct concepts.

An implementation MUST NOT claim that a particular CPU memory-ordering primitive automatically establishes application-level consistency unless the contract defines that relationship.


---

73. Serialization Fallback

A shared-memory interface MAY provide a serialized fallback.

Example:

Shared Memory
     |
     | unavailable
     v
Canonical Serialization
     |
     v
Transport

The fallback MUST preserve semantic values.

This allows implementations that do not support shared memory to remain ULABI-compatible with the same interface where the interface permits fallback.


---

74. Distributed Boundary

Shared memory MUST NOT be used to hide a distributed boundary.

If data leaves the local shared-memory domain, the implementation MUST transition to an appropriate distributed representation.

Possible mechanisms include:

serialization;

remote handles;

transport descriptors;

replicated state;

distributed shared-memory profiles.



---

75. Remote Memory

Remote memory descriptors MUST NOT be treated as ordinary local pointers.

A remote descriptor MUST expose appropriate semantics for:

latency;

failure;

availability;

consistency;

authorization.



---

76. Resource Migration

A shared-memory resource MAY migrate between memory domains.

For example:

Host Memory
     |
   migrate
     |
     v
Device Memory

Migration MUST preserve or explicitly renegotiate:

identity;

ownership;

lifetime;

permissions;

type;

synchronization;

validity.



---

77. Security Boundary Changes

Moving shared memory between domains MUST NOT silently broaden authorization.

For example:

Process Local

MUST NOT automatically become:

Host Wide

or:

Network Accessible

without explicit authorization.


---

78. Reference Invalidation

Operations that may invalidate references MUST define their invalidation behavior.

Potential invalidating operations include:

release;

resize;

remap;

migration;

revocation;

schema replacement;

generation replacement.


Affected references MUST either remain valid according to an explicit contract or fail safely.


---

79. Shared Memory API Semantics

A conceptual ULABI API MAY expose operations such as:

create_shared_memory()
map_shared_memory()
unmap_shared_memory()
grant_access()
revoke_access()
query_shared_memory()
resize_shared_memory()
synchronize_shared_memory()
create_view()
invalidate_view()
release_shared_memory()

The exact API syntax is implementation-specific.

The semantics are standardized by this specification and related profiles.


---

80. Conceptual Shared Memory Descriptor

A canonical descriptor MAY conceptually contain:

SharedMemoryDescriptor {
    resource_id
    generation
    type_id
    schema_version

    size
    alignment

    memory_domain
    locality

    ownership_model
    lifetime_model

    access_rights
    synchronization_model
    visibility_model
    consistency_model

    capability_reference

    flags
}

Implementations MAY use different physical representations.


---

81. Descriptor Validation

Before using a shared-memory descriptor, the consumer MUST validate all fields relevant to the operation.

At minimum where applicable:

resource identity;

generation;

type;

size;

alignment;

permissions;

lifetime;

domain;

synchronization;

capability.


Invalid descriptors MUST be rejected.


---

82. Descriptor Integrity

Where descriptors cross trust boundaries, implementations SHOULD protect their integrity using appropriate mechanisms.

Possible mechanisms include:

authenticated metadata;

cryptographic signatures;

MACs;

capability tokens;

secure channels.



---

83. Descriptor Replay

A stale descriptor MUST NOT regain access merely by being replayed.

Where resource authority changes over time, the implementation SHOULD use:

generation numbers;

expiration;

nonces;

capability rotation;

revocation state.



---

84. Participant Discovery

Shared-memory participants MAY be explicitly registered.

If registration is used, the contract SHOULD define:

participant identity;

authorization;

membership lifetime;

revocation;

participant failure.


Unknown participants MUST NOT receive access merely by discovering a resource identifier.


---

85. Synchronization Object Lifetime

Synchronization objects associated with a shared-memory region MUST have defined lifetimes.

A destroyed synchronization object MUST NOT be reused by stale participants as though it remained valid.


---

86. Shared Memory and Exceptions/Errors

Shared-memory operations MUST expose defined errors for failures such as:

InvalidResource
InvalidGeneration
InvalidBounds
InvalidType
InvalidAlignment
AccessDenied
ResourceExpired
ResourceRevoked
MappingFailed
SynchronizationFailed
ParticipantUnavailable
QuotaExceeded
InvalidState
UnsupportedOperation

Implementations MAY define additional errors.


---

87. Failure Atomicity

Operations that modify shared-memory metadata SHOULD be atomic with respect to their contract.

For example, a failed ownership transfer MUST NOT leave:

A owns R
B owns R

simultaneously.

A failed resize MUST NOT leave the region in an undefined partially resized state.


---

88. Recovery

Recovery behavior MUST be explicitly defined.

A recovery mechanism MAY:

1. detect failure;


2. isolate the affected state;


3. validate resource integrity;


4. restore a known state;


5. invalidate stale references;


6. re-establish synchronization;


7. resume operation.



Recovery MUST NOT silently violate ownership or authorization.


---

89. No Implicit Self-Modification

Shared-memory failures MUST NOT authorize arbitrary code modification.

If an implementation provides self-healing, it MUST follow the ULABI reliability and recovery policies.

A shared-memory failure does not itself authorize:

modify executable code
change security policy
grant capabilities
disable memory protection


---

90. Compatibility

A shared-memory implementation MUST declare which shared-memory features it supports.

Feature negotiation MAY include:

SharedMemory
ImmutableSharing
MutableSharing
AtomicOperations
LockFree
Transactions
Snapshots
PersistentMemory
DeviceMemory
ZeroCopy
CapabilityRevocation
GenerationTracking

Unsupported features MUST be reported explicitly.


---

91. Graceful Degradation

Where permitted by the interface contract:

Shared Memory
      |
unsupported
      v
Copy / Serialization

MAY be used.

The fallback MUST NOT silently change:

ownership;

lifetime;

type semantics;

security requirements;

correctness guarantees.



---

92. Conformance Requirements

A ULABI Shared Memory implementation claiming conformance MUST demonstrate, as applicable:

Resource Management

stable resource identity;

explicit lifetime;

explicit ownership;

explicit permissions;

bounds enforcement.


Safety

no unauthorized access;

no stale-resource access;

no invalid generation access;

no cross-resource corruption;

defined invalidation behavior.


Synchronization

declared synchronization model;

declared ordering;

declared visibility;

defined data-race behavior.


Security

authorization;

capability enforcement;

revocation;

isolation.


Compatibility

version validation;

schema validation;

feature negotiation;

graceful degradation where supported.



---

93. Required Conformance Tests

The shared-memory conformance suite SHOULD include at least:

SHM-001 Resource Creation
SHM-002 Resource Identity
SHM-003 Resource Mapping
SHM-004 Resource Unmapping
SHM-005 Read Permission
SHM-006 Write Permission
SHM-007 Unauthorized Access
SHM-008 Bounds Checking
SHM-009 Lifetime Expiration
SHM-010 Resource Revocation
SHM-011 Generation Validation
SHM-012 Stale Handle
SHM-013 Stale View
SHM-014 Double Release
SHM-015 Ownership Transfer
SHM-016 Concurrent Reads
SHM-017 Concurrent Writes
SHM-018 Synchronization
SHM-019 Memory Ordering
SHM-020 Visibility
SHM-021 Data Race Policy
SHM-022 Participant Crash
SHM-023 Lock Failure
SHM-024 Mapping Failure
SHM-025 Resource Quota
SHM-026 Zero-Copy
SHM-027 Copy Fallback
SHM-028 Version Compatibility
SHM-029 Schema Compatibility
SHM-030 Capability Revocation
SHM-031 Descriptor Replay
SHM-032 Cross-Process Isolation
SHM-033 Cross-Domain Isolation
SHM-034 Resize Safety
SHM-035 Migration Safety
SHM-036 Async Lifetime Safety
SHM-037 Cancellation Safety
SHM-038 Recovery
SHM-039 Fuzzed Descriptors
SHM-040 Corruption Detection


---

94. Security Testing

Security conformance SHOULD additionally test:

unauthorized mapping;

capability forgery;

capability replay;

capability escalation;

stale capability use;

revoked capability use;

descriptor manipulation;

generation confusion;

bounds bypass;

type confusion;

cross-process isolation;

cross-domain access;

resource exhaustion;

race-induced authorization bypass.



---

95. Fuzz Testing

Shared-memory descriptors, metadata and APIs SHOULD be fuzz tested.

Fuzzing SHOULD cover:

malformed descriptors;

invalid sizes;

integer overflow;

invalid generations;

invalid type IDs;

invalid permissions;

invalid synchronization modes;

truncated metadata;

corrupted headers;

race conditions.


The implementation MUST fail safely.


---

96. Formal Invariants

A conforming implementation MUST preserve these conceptual invariants:

Invariant 1 — Authorization

No Capability
    =>
No Access

Invariant 2 — Bounds

Access Range ⊆ Resource Bounds

Invariant 3 — Lifetime

Expired Resource
    =>
No Valid Access

Invariant 4 — Revocation

Revoked Capability
    =>
No Authorized Access

Invariant 5 — Generation

Old Generation
    !=
New Generation

Invariant 6 — Ownership

Exclusive Owner Count <= 1

Invariant 7 — Isolation

Invalid Access to R1
    =>
Must Not Corrupt R2

Invariant 8 — Synchronization

Declared Atomic Operation
    =>
Observed According to Atomic Contract


---

97. Reference State Machine

A shared-memory resource SHOULD conceptually follow a state machine such as:

+----------------+
                 |     Created    |
                 +-------+--------+
                         |
                       Map
                         |
                         v
                 +----------------+
                 |     Mapped     |
                 +-------+--------+
                         |
                +--------+--------+
                |                 |
             Shared            Revoke
                |                 |
                v                 v
        +---------------+   +-----------+
        |     Active    |   |  Revoked  |
        +-------+-------+   +-----+-----+
                |                 |
          Release/Expire          |
                |                 |
                v                 v
        +---------------+   +-----------+
        |   Released    |   |  Invalid  |
        +---------------+   +-----------+

Actual state transitions MUST be defined by the applicable profile.


---

98. Reference Implementation Boundary

A reference implementation SHOULD separate:

ULABI Shared Memory Contract
          |
          +-- Resource Manager
          |
          +-- Mapping Manager
          |
          +-- Permission Manager
          |
          +-- Synchronization Manager
          |
          +-- Lifetime Manager
          |
          +-- Generation Manager
          |
          +-- Validation
          |
          +-- Recovery
          |
          +-- Platform Adapter

The platform adapter MAY use operating-system-specific shared-memory facilities.

The ULABI contract MUST remain platform-neutral.


---

99. Required Separation of Concerns

The following boundaries MUST remain explicit:

Ownership
    !=
Synchronization

Lifetime
    !=
Visibility

Authorization
    !=
Mapping

Memory Safety
    !=
Performance

Zero-Copy
    !=
Correctness

Local Shared Memory
    !=
Distributed Memory

Address
    !=
Resource Identity

This separation is essential to maintaining ULABI's universal nature.


---

100. Integration Contract

This document integrates with the following ULABI specifications.

Primary dependencies

docs/abi/memory-model.md

Defines the general memory-resource model, ownership categories, lifetime model, address-space concepts, layouts, copying, views and memory representations. The shared-memory document specializes those concepts rather than redefining them.

docs/memory/memory-safety.md

Defines general memory-safety requirements including bounds safety, use-after-release, stale handles, type safety, aliasing, lifetime safety and invalid memory behavior. Shared memory MUST satisfy those requirements.

docs/memory/ownership.md

Will define authoritative ownership states and ownership transitions.

docs/memory/lifetimes.md

Defines lifetime categories, expiration and lifetime extension.

docs/memory/allocation.md

Defines allocation and resource-creation requirements.

Security dependencies

docs/security/security-model.md

Defines the general security boundary.

docs/security/capability-security.md

Defines capability semantics used for shared-memory authorization.

docs/security/sandboxing.md

Defines isolation requirements.

Compatibility dependencies

docs/compatibility/feature-negotiation.md

Defines how shared-memory capabilities are negotiated.

docs/compatibility/capability-discovery.md

Defines discovery of supported capabilities.

docs/compatibility/graceful-degradation.md

Defines fallback behavior where shared memory is unavailable.

Runtime dependencies

docs/runtime/concurrency.md

Defines general concurrency semantics.

docs/runtime/threading.md

Defines threading semantics.

docs/runtime/async-model.md

Defines asynchronous lifetime and cancellation semantics.

docs/runtime/resource-management.md

Defines generic resource lifecycle management.

Distributed dependencies

docs/distributed/distributed-abi.md

Defines the boundary between local and distributed execution.

docs/distributed/serialization.md

Defines serialization fallback.

docs/distributed/remote-calls.md

Defines remote invocation semantics.

Hardware dependencies

docs/hardware/cpu.md

Defines CPU-specific capabilities.

docs/hardware/gpu.md

Defines GPU memory interoperability.

docs/hardware/npu.md

Defines NPU memory interoperability.

docs/hardware/fpga.md

Defines FPGA memory interoperability.


---

101. What This Document Owns

docs/memory/shared-memory.md is authoritative for:

shared-memory resource semantics;

participant access;

shared-memory permissions;

shared-memory synchronization;

visibility;

shared-memory consistency;

shared-memory mapping;

shared-memory invalidation;

shared-memory generation;

shared-memory participant failure;

shared-memory reclamation requirements;

shared-memory zero-copy semantics;

shared-memory-specific conformance.


It is not authoritative for:

general ownership;

general lifetimes;

general allocation;

general memory safety;

general virtual memory;

general cryptography;

general serialization;

general distributed execution.


This ownership boundary is deliberate.


---

102. Integration Rule

Future ULABI documents MUST reference this document rather than redefining shared-memory semantics.

If another specification requires shared memory, it MUST specify:

1. which shared-memory profile it requires;


2. which access rights it requires;


3. which synchronization model it requires;


4. which lifetime model it requires;


5. which consistency guarantees it requires;


6. which security capabilities it requires.



It MUST NOT create an incompatible second shared-memory model.


---

103. Implementation Independence

ULABI shared memory MUST remain implementable by systems using:

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

Zamani;

Sankofa;

future programming languages.


No language-specific memory model may become the normative definition.


---

104. Final Principle

> ULABI shared memory defines how independently implemented participants safely share a memory resource; it does not define how any participant manages memory internally.



Shared memory is valid only when:

Identity
   +
Ownership
   +
Lifetime
   +
Authorization
   +
Bounds
   +
Synchronization
   +
Visibility
   +
Consistency
   +
Isolation
   =
Safe Shared Memory

And:

> If safe sharing cannot be established, ULABI MUST prefer explicit failure or a safe fallback over undefined shared-memory behavior.




---

What should be done under which files

Now, importantly, we should not immediately start editing every dependent file. The repository already has a dependency graph. The clean approach is to finish the independent contracts first and then integrate upward.

The current repository already contains the major memory-model files, while shared-memory.md is a placeholder.

Phase 1 — Independent foundational memory contracts

These should be completed first because other documents can depend on them without needing to redefine their semantics.

Order	File	What it must own

1	docs/memory/ownership.md	Ownership states, transfer, borrowing, sharing authority, release authority
2	docs/memory/lifetimes.md	Lifetime categories, validity intervals, expiration, extension
3	docs/memory/allocation.md	Allocation, quotas, alignment, allocation failure, deallocation
4	docs/memory/virtual-memory.md	Address spaces, mappings, protection, virtual/physical separation
5	docs/memory/shared-memory.md	The document above
6	docs/memory/memory-safety.md	Cross-cutting safety invariants over all memory mechanisms


The existing memory-safety.md already recognizes ownership, lifetime, allocation and shared-memory specifications as separate dependencies, which is exactly the architecture we should preserve.


---

Phase 2 — ABI integration

After the memory contracts are stable:

docs/abi/memory-model.md

Must integrate:

Ownership
Lifetime
Allocation
Virtual Memory
Shared Memory
Memory Safety

It should remain the general memory boundary model, not become another shared-memory specification.

docs/abi/core-abi.md

Must define which memory concepts are Core versus Profile-level.

docs/abi/data-types.md

Must define how portable types interact with:

shared buffers;

views;

alignment;

layout;

ownership;

lifetime.


docs/abi/calling-convention.md

Must define how:

borrowed buffers;

owned buffers;

transferred buffers;

shared buffers;

return values


cross function boundaries.


---

Phase 3 — Runtime integration

These files then integrate the memory subsystem:

docs/runtime/threading.md
docs/runtime/concurrency.md
docs/runtime/async-model.md
docs/runtime/resource-management.md
docs/runtime/process-model.md

Responsibilities:

threading.md

Thread-level access and synchronization.

concurrency.md

General concurrent execution semantics.

async-model.md

Shared-memory lifetime during asynchronous execution.

resource-management.md

Generic lifecycle integration.

process-model.md

Cross-process mappings and isolation.


---

Phase 4 — Security integration

These files should consume the shared-memory contract rather than reinvent it:

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md

The critical rule is:

Shared Memory
      |
      v
Security Policy
      |
      v
Capability
      |
      v
Authorized Mapping

Not:

Shared Memory
      |
      v
automatic access


---

Phase 5 — Distributed integration

These files should establish where shared memory ends and distributed interoperability begins:

docs/distributed/distributed-abi.md
docs/distributed/serialization.md
docs/distributed/remote-calls.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

The most important architectural rule is:

> Do not make distributed memory look identical to local shared memory.



The distinction between locality and remote execution is already part of the ULABI architecture.


---

Phase 6 — Hardware integration

The hardware layer should implement shared-memory semantics through platform profiles:

docs/hardware/cpu.md
docs/hardware/gpu.md
docs/hardware/npu.md
docs/hardware/fpga.md
docs/hardware/quantum.md
docs/hardware/future-architectures.md

Especially:

CPU
 |
 +---- Host Memory
 |
 +---- Shared Memory
 |
 +---- Device Memory
 |
 +---- Accelerator Memory

The universal semantics remain in ULABI; hardware documents define platform-specific capabilities.


---

Phase 7 — Compatibility

These should then consume the finalized shared-memory capability set:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

For example:

Implementation A
      |
      | supports
      v
SharedMemory
ImmutableSharing
ZeroCopy
GenerationTracking
      |
      v
Implementation B

If B does not support shared memory:

SharedMemory
     |
 unsupported
     v
Canonical Copy


---

Phase 8 — Conformance

Finally:

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/test-suite.md
docs/standards/certification.md
docs/standards/reference-implementations.md

Shared memory becomes its own conformance capability:

ULABI Memory
    |
    +-- Memory Model
    +-- Ownership
    +-- Lifetimes
    +-- Allocation
    +-- Memory Safety
    +-- Virtual Memory
    +-- Shared Memory


---

Required code/module architecture

The documentation should be completed before committing to a specific implementation language. The code should implement the contracts, not define them.

I recommend this implementation-neutral module architecture.

implementations/
│
├── core/
│   ├── identity/
│   ├── types/
│   ├── descriptors/
│   ├── errors/
│   ├── versioning/
│   └── capabilities/
│
├── memory/
│   ├── resource/
│   ├── ownership/
│   ├── lifetime/
│   ├── allocation/
│   ├── safety/
│   ├── virtual_memory/
│   ├── shared_memory/
│   ├── mapping/
│   ├── views/
│   ├── generation/
│   ├── permissions/
│   └── reclamation/
│
├── synchronization/
│   ├── atomics/
│   ├── mutex/
│   ├── rwlock/
│   ├── barriers/
│   ├── fences/
│   ├── lock_free/
│   └── transactions/
│
├── runtime/
│   ├── threading/
│   ├── concurrency/
│   ├── async/
│   ├── resources/
│   └── processes/
│
├── security/
│   ├── authorization/
│   ├── capabilities/
│   ├── revocation/
│   └── sandbox/
│
├── transport/
│   ├── local/
│   ├── ipc/
│   ├── shared_memory/
│   └── distributed/
│
├── serialization/
│   ├── canonical/
│   ├── descriptors/
│   └── fallback/
│
├── hardware/
│   ├── cpu/
│   ├── gpu/
│   ├── npu/
│   ├── fpga/
│   └── accelerator/
│
└── validation/
    ├── descriptors/
    ├── memory/
    ├── synchronization/
    ├── security/
    └── conformance/

Shared-memory-specific code modules

The eventual shared-memory implementation should have, at minimum:

shared_memory/
├── resource
├── descriptor
├── manager
├── mapper
├── permissions
├── participant
├── ownership
├── lifetime
├── generation
├── synchronization
├── visibility
├── consistency
├── reclamation
├── revocation
├── validation
├── recovery
└── errors

And platform adapters:

shared_memory/platform/
├── posix
├── linux
├── windows
├── macos
├── wasm
├── android
├── ios
├── embedded
└── accelerator

These should be adapters, not the ULABI semantic definition.


---

Final dependency order

The cleanest implementation/documentation order is therefore:

1. ownership.md
        ↓
2. lifetimes.md
        ↓
3. allocation.md
        ↓
4. virtual-memory.md
        ↓
5. shared-memory.md
        ↓
6. memory-safety.md
        ↓
7. abi/memory-model.md
        ↓
8. core-abi.md
        ↓
9. data-types.md
        ↓
10. calling-convention.md
        ↓
11. runtime/resource-management.md
        ↓
12. runtime/concurrency.md
        ↓
13. runtime/threading.md
        ↓
14. runtime/async-model.md
        ↓
15. security model
        ↓
16. distributed model
        ↓
17. hardware profiles
        ↓
18. compatibility
        ↓
19. conformance
        ↓
20. reference implementation
        ↓
21. independent implementations

This ordering gives you the property you asked for: when one foundational document is declared complete, later documents are consumers of its contract rather than reasons to reopen and redesign it.

Most importantly, shared-memory.md now has a defined ownership boundary: it specifies how shared memory behaves, while ownership.md specifies who owns, lifetimes.md specifies how long, allocation.md specifies how memory is created, memory-safety.md specifies what safety means, and security specifies who is authorized. That separation prevents the ULABI specification from becoming a collection of overlapping documents.