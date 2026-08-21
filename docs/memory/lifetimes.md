I read the repository first rather than drafting lifetimes.md in isolation.

The current architecture already establishes the important boundary: ULABI is language/runtime/platform neutral, with explicit ownership and lifetime semantics, safe zero-copy, capability security, process/distributed modes, deterministic representations, and implementation independence.

I also checked the surrounding memory specifications. docs/abi/memory-model.md already defines the high-level memory-resource, ownership, borrowing, transfer, invalidation, and lifetime concepts, while docs/memory/allocation.md defines allocation/deallocation semantics.   Meanwhile, docs/memory/ownership.md, docs/memory/memory-safety.md, and the requested docs/memory/lifetimes.md are currently placeholders.

So lifetimes.md should not repeat the memory model or allocator specification. It should become the authoritative specification for validity duration, expiration, extension, borrowing intervals, revocation, transfer, destruction, asynchronous retention, views, and lifetime violations.

Below is the complete document.


---

docs/memory/lifetimes.md

# ULABI Lifetime Specification

Universal Language Application Binary Interface

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Repository:** `Benwellonedge28/ULABI`  
**Document:** `docs/memory/lifetimes.md`  
**Specification Area:** Memory / Lifetimes  
**Status:** Architecture / Design Specification  
**Specification Version:** 0.2.0-draft  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI lifetime contract.

The purpose of the lifetime model is to define:

- when a ULABI resource becomes valid;
- when it becomes invalid;
- how long a reference may be used;
- how ownership affects validity;
- how borrowing affects validity;
- how lifetime extension works;
- how lifetime expiration is represented;
- how resources are revoked;
- how asynchronous operations retain resources;
- how views retain underlying resources;
- how transferred resources change validity;
- how implementations detect lifetime violations;
- how lifetime failures are reported;
- how independent language implementations preserve lifetime safety across ULABI boundaries.

ULABI lifetime semantics apply to resources including:

- memory;
- buffers;
- records;
- objects;
- handles;
- views;
- streams;
- asynchronous operations;
- shared resources;
- device resources;
- accelerator resources;
- process resources;
- runtime-managed resources.

ULABI does not require a particular memory-management strategy.

An implementation may use:

- manual memory management;
- garbage collection;
- reference counting;
- ownership and borrowing;
- arenas;
- regions;
- pools;
- tracing;
- runtime handles;
- hardware protection;
- capability systems;
- another mechanism.

ULABI specifies the observable lifetime contract rather than the internal implementation mechanism.

---

# 2. Relationship to the ULABI Architecture

`ULABI-DESIGN.md` establishes:

- language neutrality;
- runtime neutrality;
- platform neutrality;
- architecture neutrality;
- implementation independence;
- explicit ownership boundaries;
- safe memory interoperability;
- zero-copy interoperability where safe;
- deterministic behavior;
- capability security;
- failure-oriented design.

This document specializes those principles for lifetime semantics.

The relationship is:

```text
                    ULABI
                      |
              Memory Contract
                      |
               Ownership Model
                      |
               Lifetime Contract
                      |
       +--------------+--------------+
       |              |              |
    Validity       Borrowing      Transfer
       |              |              |
       +--------------+--------------+
                      |
                Implementation

The lifetime specification therefore depends conceptually on the memory model and ownership model but does not replace either.


---

3. Normative Language

The terms:

MUST

MUST NOT

REQUIRED

SHALL

SHALL NOT

SHOULD

SHOULD NOT

MAY


are normative.

"MUST" means a conforming implementation is required to satisfy the requirement.

"SHOULD" means the behavior is strongly recommended unless an explicitly documented implementation constraint prevents it.

"MAY" means the behavior is optional.


---

4. Non-Goals

This specification does not:

1. mandate garbage collection;


2. mandate reference counting;


3. mandate compile-time lifetime checking;


4. mandate Rust-style borrowing;


5. mandate manual memory management;


6. mandate a particular allocator;


7. define physical memory management;


8. define virtual-memory management;


9. define processor page management;


10. define operating-system process management;


11. define serialization;


12. define distributed consensus;


13. define one universal object model;


14. require pointers to be portable;


15. require all resources to have identical lifetime semantics;


16. make remote resources behave like local memory;


17. hide lifetime boundaries for convenience.




---

5. Fundamental Principle

> A ULABI reference is valid only for the lifetime explicitly granted by its contract.



Holding a reference, pointer, handle, or wrapper does not by itself extend the lifetime of the referenced resource.

Conceptually:

Resource
   |
   +---- valid ----+
                   |
                   v
              expiration
                   |
                   v
                invalid

A conforming implementation MUST NOT silently extend a lifetime merely because a consumer still possesses a reference.


---

6. Lifetime and Ownership Are Different

Ownership answers:

> Who is responsible for the resource?



Lifetime answers:

> During which interval is access to the resource valid?



These concepts are related but not identical.

Example:

Owner A
   |
   | owns
   v
Resource R
   |
   | borrow
   v
Consumer B

A owns the resource.

B receives only a bounded lifetime.

Therefore:

ownership(R) = A
lifetime(B's reference) = defined borrow interval

The borrower's lifetime MUST NOT automatically become the owner's lifetime.


---

7. Lifetime States

A ULABI resource SHOULD be modeled using the following semantic states:

Uncreated
    |
    v
Created
    |
    v
Valid
    |
    +--------+
    |        |
    v        v
Transferred  Revoked
    |        |
    v        |
Valid        |
    |        |
    +--------+
         |
         v
      Expired
         |
         v
      Released
         |
         v
       Invalid

An implementation MAY use a different internal state machine.

The externally observable semantics MUST remain compatible with this model.


---

8. Core Lifetime States

8.1 Uncreated

The resource does not yet exist.

Access MUST fail.

8.2 Created

The resource has been successfully created but may not yet be exposed to another component.

8.3 Valid

The resource may be accessed according to its permissions and lifetime contract.

8.4 Transferred

Ownership has moved to another authorized component.

The previous owner MUST no longer exercise ownership privileges.

8.5 Revoked

Access has been explicitly invalidated before the originally expected expiration.

8.6 Expired

The permitted lifetime has ended.

8.7 Released

The owning contract has ended the resource.

8.8 Invalid

The resource MUST NOT be accessed through the invalidated reference or capability.


---

9. Lifetime Interval

Every lifetime-bearing resource has a semantic validity interval.

Conceptually:

creation ---------------------- expiration
   |                                 |
   +----------- VALID ---------------+

The contract MUST define whether the boundaries are:

inclusive;

exclusive;

event-based;

operation-based.


For ordinary resource lifetimes, the recommended model is:

valid_from <= access < valid_until

Implementations MAY use different internal clocks or mechanisms provided that the externally observable semantics remain correct.


---

10. Lifetime Categories

ULABI supports the following conceptual lifetime categories.

10.1 CallLifetime

Valid only during a defined call.

call begins
    |
    v
resource valid
    |
call returns
    |
    v
resource invalid

10.2 ScopeLifetime

Valid within a defined logical scope.

10.3 BorrowLifetime

Valid for the duration of an explicit borrow.

10.4 ObjectLifetime

Valid while the owning object remains alive.

10.5 SessionLifetime

Valid while an established session remains active.

10.6 ProcessLifetime

Valid while the owning process remains active.

10.7 ExplicitLifetime

Valid until an explicit release, revoke, or expiration event.

10.8 ReferenceLifetime

Valid while a defined reference-retention mechanism remains active.

10.9 ResourceLifetime

Valid according to the resource's own lifecycle contract.


---

11. Lifetime Contract

A lifetime contract SHOULD contain:

LifetimeContract {
    lifetime_id
    resource_id
    owner
    creation_condition
    validity_condition
    expiration_condition
    release_condition
    revocation_condition
    extension_policy
    transfer_policy
    borrow_policy
    retention_policy
    failure_policy
}

The exact binary representation is implementation-defined unless standardized by another ULABI schema.


---

12. Lifetime Identity

Where lifetime tracking is required, an implementation SHOULD provide a stable semantic lifetime identity.

Conceptually:

LifetimeIdentity {
    resource_id
    generation
    contract_id
}

This allows stale references to be distinguished from current resources.

A raw pointer MUST NOT be considered sufficient lifetime identity for portable ULABI semantics.


---

13. Creation

A lifetime begins only after successful resource creation.

Example:

allocate()
   |
   +-- failure --> no valid resource
   |
   +-- success --> valid resource

An implementation MUST NOT expose a resource as valid before successful creation.

If creation fails, no lifetime may be assumed to exist.


---

14. Publication

A resource may be created before being exposed to another component.

Publication MUST establish:

resource identity;

access permissions;

lifetime;

ownership;

applicable capabilities.


A resource MUST NOT be published without sufficient information for the receiving component to determine whether it may legally retain or access it.


---

15. Borrowing

Borrowing grants temporary access without transferring ownership.

Example:

Owner A
   |
   | owns
   v
Resource R
   |
   | borrow [t1, t2)
   v
Consumer B

The borrow contract MUST define:

borrower;

owner;

permissions;

start;

end;

whether mutation is allowed;

whether nested borrows are allowed;

whether retention is allowed;

whether transfer is allowed.


A borrower MUST NOT access the resource after the borrow lifetime expires.


---

16. Borrow Lifetime Cannot Be Assumed Infinite

The following is invalid:

borrow(resource)
store_reference_forever()

unless the contract explicitly grants an indefinite lifetime.

A consumer MUST NOT infer:

reference exists => resource remains valid


---

17. Lifetime Extension

A consumer requiring continued access beyond the original lifetime MUST use an explicitly supported extension mechanism.

Possible mechanisms include:

copy;

retain;

ownership transfer;

promotion;

materialization;

reference acquisition;

resource duplication.


Example:

CallLifetime
     |
     | retain
     v
SessionLifetime

Lifetime extension MUST be authorized by the resource contract.


---

18. Copy as Lifetime Extension

A copy may create a new independently owned resource.

Example:

Temporary Input
      |
      | copy
      v
Independent Resource

The new resource has its own lifetime.

The original resource may expire without invalidating the copy.


---

19. Retention

A resource may support explicit retention.

Conceptually:

retain(resource)

Successful retention creates or extends an authorized lifetime.

Retention MUST NOT occur implicitly unless the contract explicitly specifies automatic retention.

If retention fails, the consumer MUST NOT assume that the lifetime was extended.


---

20. Promotion

A short-lived resource may be promoted to a longer-lived resource.

Example:

CallLifetime
     |
     | promote
     v
ObjectLifetime

Promotion MUST be explicit.

Promotion may involve:

copying;

moving;

reference registration;

allocator transfer;

ownership transfer;

runtime bookkeeping.



---

21. Lifetime Expiration

A lifetime expires when its expiration condition becomes true.

Expiration conditions may include:

call return;

scope exit;

timeout;

session termination;

process termination;

explicit release;

ownership transition;

capability revocation;

resource shutdown;

policy decision;

device failure.


After expiration, the expired reference MUST NOT be treated as valid.


---

22. Explicit Release

Explicit release ends a resource lifetime when authorized.

Example:

release(resource)

After successful release:

resource = invalid

The previous owner MUST NOT continue using the resource.


---

23. Revocation

Revocation terminates access before normal expiration.

Example:

Valid
  |
  | revoke
  v
Revoked

Revocation may be required for:

security policy changes;

capability removal;

session termination;

device removal;

fault isolation;

resource exhaustion;

administrative control.


Revocation MUST take precedence over ordinary retention.


---

24. Revocation Is Not Release

Revocation means:

> Access is no longer authorized.



Release means:

> The owning resource lifecycle has ended.



A revoked resource may remain physically allocated for cleanup, rollback, or recovery.

Therefore:

revoked != necessarily deallocated


---

25. Lifetime Invalidation

A lifetime becomes invalid when any mandatory validity condition is no longer satisfied.

Possible causes:

expiration
release
revocation
ownership transfer
process termination
session termination
capability removal
resource failure
security isolation

An implementation MUST NOT continue exposing invalid resources as valid.


---

26. Ownership Transfer

Ownership transfer changes responsibility for the resource.

Example:

Before:

A owns R


Transfer


After:

B owns R
A does not own R

The transition MUST establish a clear lifetime boundary.

After successful transfer, the previous owner MUST NOT use the resource through its former ownership rights.


---

27. Transfer Failure

If an ownership transfer fails, the original ownership and lifetime MUST remain valid unless the contract explicitly specifies another atomic transition.

The preferred model is:

Old Owner
    |
 transfer attempt
    |
 +--+--+
 |     |
fail  success
 |     |
 v     v
Old   New
state owner
remains
valid


---

28. Lifetime of Returned Values

A returned value MUST define its lifetime semantics.

Possible models include:

ReturnedByValue
BorrowedReturn
TransferredReturn
HandleReturn
SharedReturn

A caller MUST NOT assume that a returned pointer or reference remains valid after the call unless the contract explicitly says so.


---

29. Borrowed Return Values

A function may return a borrowed value.

Example:

get_view() -> Borrowed<T>

The contract MUST specify:

owner;

validity interval;

whether the caller may retain;

whether the caller may mutate;

whether the caller may transfer.


A borrowed return MUST NOT silently become an owned return.


---

30. Transferred Return Values

A function may return ownership.

Example:

create_buffer() -> Owned<Buffer>

The caller becomes responsible for the resulting lifetime according to the ownership contract.


---

31. Asynchronous Operations

Asynchronous calls create special lifetime requirements.

Example:

Caller
   |
   | submit(buffer)
   v
Async Operation
   |
   | executes later
   v
Worker

If an asynchronous operation retains a resource, the contract MUST explicitly state that retention.

The caller MUST NOT release the resource while the operation still legitimately depends on it.


---

32. Async Retention

An async operation may establish:

CallLifetime
      |
      | retain for operation
      v
OperationLifetime

The retention MUST end when the operation no longer requires the resource.

Example:

submit()
   |
   v
Running
   |
   v
Completed
   |
   v
Retention ends


---

33. Async Cancellation

Cancellation MUST define what happens to retained resources.

Possible semantics:

Cancel requested
      |
      v
Stopping
      |
      v
Cleanup
      |
      v
Retention released

Cancellation MUST NOT automatically imply that underlying resources become invalid unless the contract explicitly defines that behavior.


---

34. Failure of Asynchronous Operations

If an async operation fails, all retained resources MUST transition according to the operation's failure contract.

The implementation MUST NOT leak lifetime obligations merely because the operation failed.


---

35. Memory Views

A memory view is dependent on an underlying resource.

Example:

Underlying Resource R
        |
        +---- View A
        |
        +---- View B

A view MUST NOT outlive its underlying resource unless the view has been materialized or otherwise converted into an independent resource.


---

36. View Retention

A view MAY retain the underlying resource.

If it does:

View lifetime
      |
      +---- keeps R valid

The retention mechanism MUST be explicit.

Destroying the view SHOULD release its retention obligation.


---

37. Slices

A slice is a bounded view into another resource.

A slice MUST define:

source resource;

offset;

length;

access permissions;

lifetime;

ownership relationship.


A slice MUST NOT permit access beyond its declared bounds.


---

38. Containers and Nested Lifetimes

A container may own or borrow nested resources.

Example:

Container
   |
   +-- Element A
   +-- Element B
   +-- Element C

The lifetime relationship between the container and nested elements MUST be explicit.

Possible relationships include:

owned-by-container;

borrowed-from-container;

independently-owned;

shared;

externally-owned.


Destroying a container MUST NOT unexpectedly invalidate externally-owned resources.


---

39. Object Graphs

For object graphs:

A -> B -> C

ULABI must distinguish:

ownership edges;

borrowing edges;

sharing edges;

weak references;

external-resource edges.


A graph MUST NOT be assumed to have one universal destruction strategy.


---

40. Cycles

If a lifetime mechanism uses ownership graphs, cycles may occur.

Example:

A -> B
^    |
|____|

An implementation using reference counting MUST provide an appropriate mechanism for cycles, such as:

weak references;

cycle detection;

explicit release;

tracing;

another defined mechanism.


ULABI does not mandate the mechanism.

The implementation MUST ensure that a cycle cannot cause unbounded resource retention where the resource contract requires eventual release.


---

41. Weak References

A weak reference does not keep a resource alive.

Conceptually:

Owner ----strong----> Resource
Observer --weak-----> Resource

After resource expiration:

strong reference = invalid
weak reference = expired

A weak reference MUST be safely detectable as expired.


---

42. Resource Handles

Opaque handles may represent resources whose lifetime is controlled separately from their representation.

A handle MUST have defined validity semantics.

A stale handle MUST NOT silently refer to an unrelated resource.

Generation counters or equivalent protection SHOULD be used where handles can be reused.


---

43. Generation Protection

Example:

resource_id = 42
generation = 7

After release:

resource_id = 42
generation = 8

A handle referencing generation 7 MUST NOT gain access to generation 8.

This prevents stale references from becoming valid again through resource-ID reuse.


---

44. Pointer Lifetimes

A pointer crossing a ULABI boundary MUST have an explicitly defined lifetime.

The following assumption is invalid:

pointer received
     |
     v
pointer remains valid forever

A pointer MUST be classified as, for example:

borrowed;

owned;

shared;

opaque;

address-local;

handle-backed.



---

45. Pointer Lifetime Across Processes

A raw pointer MUST NOT automatically survive a process boundary.

For example:

Process A:
0x10000000

does not imply:

Process B:
0x10000000

refers to the same resource.

Cross-process lifetime must use an explicit mechanism such as:

shared-memory descriptor;

operating-system handle;

IPC resource;

serialized resource;

ULABI capability;

another standardized mechanism.



---

46. Distributed Lifetimes

Remote resources have failure-prone lifetimes.

Therefore:

LocalLifetime != automatically RemoteLifetime

A distributed lifetime may be terminated by:

network failure;

remote process failure;

session expiration;

lease expiration;

capability revocation;

remote shutdown.


Remote lifetime semantics MUST be defined by the applicable distributed profile.


---

47. Leases

A distributed implementation MAY use leases.

Example:

Lease acquired
     |
     v
Resource valid
     |
 renewal
     |
     v
Resource remains valid
     |
 lease expires
     |
     v
Resource invalid

Lease renewal MUST be explicit.

A failed renewal MUST NOT silently extend the lifetime.


---

48. Time-Based Lifetimes

Time-based lifetimes MUST define:

time source;

units;

precision;

clock semantics;

expiration rule;

clock failure behavior.


For distributed systems, implementations SHOULD avoid relying solely on unsynchronized wall clocks.


---

49. Session Lifetimes

A resource may remain valid while a session exists.

Example:

Session created
     |
     v
Resource valid
     |
session active
     |
     v
Session terminated
     |
     v
Resource invalid

Session termination MUST trigger the defined resource cleanup or revocation behavior.


---

50. Process Lifetimes

A process-local resource may have:

ProcessLifetime

When the process terminates:

process terminated
       |
       v
process-scoped resources expire

The implementation MUST define whether resources are:

automatically released;

transferred;

persisted;

revoked;

recovered.



---

51. Device Lifetimes

Device-backed resources may disappear independently of ordinary memory.

Examples:

GPU context;

accelerator buffer;

device mapping;

DMA resource.


A device failure MUST transition dependent resources into a defined invalid, failed, or recoverable state.


---

52. Persistent Lifetimes

Persistent resources may survive:

process termination;

machine restart;

runtime restart.


Persistence MUST be explicitly declared.

A resource MUST NOT be treated as persistent merely because its underlying bytes happen to exist after process termination.


---

53. Recovery and Lifetime

A recoverable resource may transition:

Valid
  |
failure
  v
Unavailable
  |
recovery
  v
Valid

If recovery creates a replacement resource, the old resource identity MUST NOT automatically become valid again.

The replacement MUST have a defined identity and lifetime relationship.


---

54. Rollback

If an operation modifies a resource and subsequently fails, rollback semantics MUST specify which lifetime state applies.

Example:

Valid
  |
mutation
  v
Tentative
  |
 failure
  v
Rollback
  |
  v
Valid

A failed transaction MUST NOT leave a resource simultaneously marked valid and invalid.


---

55. Capability Revocation

If access is controlled by a capability, revocation of that capability MUST invalidate the associated access rights.

Capability revocation does not necessarily deallocate the underlying resource.

Therefore:

capability revoked
       |
       v
access invalid
       |
       +---- resource may still exist


---

56. Lifetime and Security

Lifetime violations can become security vulnerabilities.

Implementations MUST protect against:

use-after-free;

stale handles;

use-after-transfer;

expired capabilities;

lifetime confusion;

unauthorized retention;

unauthorized release;

cross-domain lifetime confusion.


Lifetime checks SHOULD occur before security-sensitive access.


---

57. Lifetime and Memory Safety

Lifetime correctness is one component of memory safety.

The lifetime specification defines:

WHEN access is valid

The memory-safety specification defines broader:

WHETHER access is safe

The two documents MUST remain separate.


---

58. Lifetime and Allocation

Allocation creates resources.

Lifetime determines how long they remain valid.

Therefore:

Allocation
    |
    v
Resource created
    |
    v
Lifetime begins
    |
    v
Lifetime ends
    |
    v
Deallocation

The allocation specification defines allocation and release mechanics.

This specification defines validity intervals.


---

59. Lifetime and Ownership

Ownership determines responsibility.

Lifetime determines validity.

An implementation MUST NOT use ownership terminology to hide lifetime behavior.

For example:

"owned"

does not automatically mean:

valid forever


---

60. Lifetime and Zero-Copy

Zero-copy interfaces require especially strict lifetime contracts.

Example:

Producer Buffer
       |
       | borrow
       v
Consumer

The producer MUST keep the buffer valid for the entire authorized borrow interval.

The consumer MUST NOT retain the buffer beyond that interval unless explicit retention is supported.


---

61. Lifetime and Shared Memory

Shared memory requires explicit lifetime management.

A shared region MUST define:

creator;

participants;

access rights;

synchronization;

lifetime;

revocation;

destruction;

failure behavior.


Shared access MUST NOT imply indefinite lifetime.


---

62. Lifetime and Concurrency

Concurrent access requires a defined lifetime relationship.

A resource MUST remain valid for every operation that is authorized to access it.

If one participant may release a resource while another is accessing it, the interface MUST provide an explicit synchronization or retention mechanism.

This prevents:

Thread A: access R
Thread B: release R
Thread A: continue access R

unless the contract explicitly permits such behavior.


---

63. Atomic Lifetime Transitions

Where multiple participants may concurrently affect lifetime, transitions SHOULD be atomic from the perspective of the ULABI contract.

Examples:

retain;

release;

transfer;

revoke;

acquire;

lease renewal.


The implementation MAY use locks, atomics, reference counts, capabilities, transactions, or other mechanisms.


---

64. Lifetime Race Conditions

Implementations MUST prevent undefined lifetime races at the ULABI boundary.

Example:

A checks resource valid
B releases resource
A accesses resource

A validity check MUST NOT become meaningless before the corresponding authorized access when atomicity is required.


---

65. Lifetime Tokens

An implementation MAY expose an opaque lifetime token.

Conceptually:

LifetimeToken {
    resource_id
    generation
    contract_id
    permissions
}

The token may be required to access or retain a resource.

Tokens MUST NOT be forgeable where security requires unforgeability.


---

66. Lifetime Validation

A validator SHOULD be able to determine:

resource validity;

ownership status;

lifetime category;

expiration;

revocation;

transfer state;

retention state;

generation;

applicable permissions.


Validation failures MUST be deterministic enough for conformance testing.


---

67. Lifetime Errors

Implementations SHOULD expose standardized semantic errors such as:

LifetimeExpired
LifetimeRevoked
ResourceReleased
ResourceInvalid
BorrowExpired
OwnershipTransferred
UnauthorizedRetention
UnauthorizedRelease
StaleHandle
GenerationMismatch
LeaseExpired
SessionClosed
ProcessTerminated
DeviceUnavailable
LifetimeConflict
InvalidLifetimeContract

The exact error encoding belongs to the ULABI error model.


---

68. Error Ordering

When multiple failures apply, implementations SHOULD use a deterministic precedence.

Recommended precedence:

Invalid contract
      |
      v
Authorization
      |
      v
Resource identity
      |
      v
Lifetime validity
      |
      v
Access permission
      |
      v
Operation-specific validation

The final normative error ordering belongs to the core error specification.


---

69. No Silent Lifetime Extension

An implementation MUST NOT silently convert:

CallLifetime

into:

SessionLifetime

or:

ProcessLifetime

merely because doing so is convenient.

Lifetime changes must be explicit.


---

70. No Silent Lifetime Reduction

Likewise, an implementation MUST NOT silently reduce a contract.

For example:

requested: SessionLifetime
provided: CallLifetime

must not be reported as successful unless the contract explicitly permits the downgrade and the caller accepts it.


---

71. Lifetime Negotiation

If lifetime guarantees depend on implementation capabilities, the participants MAY negotiate.

Example:

Consumer:
requires SessionLifetime

Provider:
supports CallLifetime only

Result:
incompatible

The provider MUST NOT falsely claim compliance.

Feature negotiation belongs to:

docs/compatibility/feature-negotiation.md


---

72. Graceful Degradation

If a longer lifetime is unavailable, an implementation MAY offer a safe alternative.

Example:

Borrow for SessionLifetime
        |
        unavailable
        |
        v
Copy into caller-owned resource

The fallback MUST preserve semantic correctness.

It MUST NOT silently alter ownership or validity.


---

73. Lifetime Across Language Boundaries

Different languages may map the same lifetime contract differently.

Example:

Language A:
manual ownership

Language B:
garbage collection

Language C:
reference counting

Language D:
borrow checker

             |
             v
          ULABI
             |
             v

same semantic lifetime contract

ULABI defines the boundary semantics, not the implementation mechanism.


---

74. Garbage-Collected Languages

A garbage-collected implementation MUST NOT assume that garbage collection automatically satisfies an external ULABI lifetime contract.

If a resource must remain valid:

GC reachability

may be insufficient unless the ULABI contract explicitly maps reachability to retention.

External resources may require:

explicit retention;

pinning;

handles;

finalization;

resource registration.



---

75. Reference-Counted Languages

Reference counting MAY implement lifetime retention.

However:

reference exists

must correspond to a valid ULABI retention contract.

Reference cycles MUST be handled according to the implementation's memory-management model.


---

76. Ownership-Based Languages

Ownership and borrowing systems MAY provide compile-time enforcement.

The compiler MAY reject programs that cannot prove lifetime validity.

This is an implementation technique, not a requirement imposed on all ULABI languages.


---

77. Manual-Memory Languages

Manual-memory implementations MUST provide sufficient runtime or API mechanisms to satisfy the ULABI lifetime contract.

Manual control MUST NOT permit the implementation to report an invalid resource as valid.


---

78. Automatic Resource Management

Languages may provide:

destructors;

finalizers;

RAII;

defer mechanisms;

scoped cleanup.


These may implement lifetime transitions.

However, nondeterministic finalization MUST NOT be treated as deterministic release where ULABI requires deterministic release.


---

79. Finalization

Finalization is not automatically equivalent to explicit release.

If a ULABI resource requires deterministic release, the implementation MUST provide a deterministic mechanism.

Garbage collection timing MUST NOT be used as a substitute for a required deterministic lifetime transition.


---

80. Lifetime During Exceptions

If an exception, trap, cancellation, or abnormal control-flow event occurs, lifetime behavior MUST remain defined.

The implementation MUST specify which resources:

remain valid;

are released;

are transferred;

are revoked;

require recovery.



---

81. Lifetime During Process Failure

When a process terminates unexpectedly:

Process failure
      |
      v
Process-scoped resources

must follow their declared recovery or destruction policy.

Resources shared with other processes MUST NOT automatically become invalid unless the contract says so.


---

82. Lifetime During Network Failure

Network failure MUST NOT be interpreted as successful resource release.

A distributed implementation must distinguish:

remote unavailable

from:

resource successfully released

unless the distributed protocol guarantees equivalence.


---

83. Lifetime During Device Failure

Device failure may invalidate device-backed resources.

The implementation MUST report the resulting state.

A resource MUST NOT silently remain usable after the underlying device has become unavailable.


---

84. Nested Lifetimes

A resource may depend on another resource.

Example:

Session
   |
   +-- Connection
          |
          +-- Buffer

The dependent resource MUST NOT outlive a required parent resource unless it has been detached or promoted.


---

85. Parent-Child Lifetime Rule

If:

Child depends on Parent

then:

Parent expires

must either:

1. invalidate the child;


2. detach the child;


3. transfer the child;


4. materialize the child;


5. follow another explicitly defined recovery policy.



It MUST NOT silently leave the child in an undefined state.


---

86. Independent Lifetime

A resource with an independent lifetime may outlive its creator.

Example:

Creator
   |
   | creates
   v
Independent Resource
   |
creator ends
   |
   v
resource remains valid

This MUST be explicitly specified.


---

87. Lifetime Domains

Implementations MAY define lifetime domains such as:

CallDomain
ThreadDomain
TaskDomain
ProcessDomain
SessionDomain
DeviceDomain
HostDomain
PersistentDomain
DistributedDomain

A resource MUST identify the applicable domain when domain semantics affect validity.


---

88. Lifetime Domain Transfer

Moving a resource between domains MUST define the lifetime transition.

Example:

ProcessDomain
      |
      | transfer
      v
SessionDomain

The transfer MUST specify:

whether identity changes;

whether ownership changes;

whether permissions change;

whether lifetime changes;

whether existing references remain valid.



---

89. Lifetime Identity Preservation

When a resource is moved without semantic replacement, its identity SHOULD remain stable.

When a new resource is created as a replacement, the new resource MUST have a distinguishable identity.

This prevents:

old resource
    |
failure
    |
new resource

from being incorrectly treated as the same resource without an explicit replacement relationship.


---

90. Replacement Resources

A recovery mechanism MAY replace an invalid resource.

Example:

Resource A
    |
 failure
    v
Replacement B

The contract SHOULD expose:

replacement_of = A

where such lineage is useful.

B does not automatically inherit A's identity or all of A's capabilities.


---

91. Lifetime and Transactions

Transactions MAY establish temporary lifetimes.

Example:

Begin
  |
  v
Tentative resources
  |
  +---- Commit ----> durable/normal lifetime
  |
  +---- Abort -----> resources expire

Transaction lifetime rules MUST be explicit.


---

92. Lifetime and Streams

Streams may produce values whose lifetimes differ from the stream itself.

The stream contract MUST specify whether produced values are:

borrowed;

owned;

copied;

retained;

valid until next item;

valid until stream close;

independently valid.


A consumer MUST NOT assume that all streamed values have the same lifetime.


---

93. Iterator Lifetimes

An iterator may return borrowed elements.

Example:

iterator.next()
      |
      v
Borrowed<Element>

The lifetime of the element MUST be defined.

An implementation MAY invalidate the previous element when the iterator advances if the contract explicitly specifies:

valid_until_next()


---

94. Callback Lifetimes

A callback may receive borrowed resources.

The callback MUST NOT retain them beyond the permitted callback lifetime unless explicit retention is available.

If retention is required, the callback contract MUST expose the required operation.


---

95. Event Lifetimes

Events may carry references or handles.

The event contract MUST specify whether payload data remains valid:

during callback;

until acknowledgment;

until event completion;

for a defined retention period;

independently of the event.



---

96. Lifetime and ABI Calls

The calling convention MUST identify whether parameters and return values are:

borrowed;

owned;

transferred;

copied;

shared;

handle-based.


The calling convention MUST NOT leave lifetime semantics implicit where the distinction affects correctness.


---

97. Lifetime Metadata

A ULABI interface descriptor SHOULD be able to describe lifetime metadata.

Conceptually:

ParameterLifetime {
    mode
    owner
    borrow_duration
    retention_allowed
    transfer_allowed
    mutation
}

The exact descriptor schema belongs to the type/interface descriptor specifications.


---

98. Lifetime Contracts in Function Signatures

Conceptually:

function process(
    input: Borrowed<Buffer, CallLifetime>
) -> Owned<Result>

or:

function retain(
    input: Borrowed<Buffer>
) -> Owned<BufferHandle>

The representation is illustrative.

The semantic information is normative.


---

99. Lifetime Mismatch

A call is incompatible if the consumer requires a lifetime longer than the provider guarantees.

Example:

Provider:
CallLifetime

Consumer:
SessionLifetime

This MUST NOT be treated as compatible without:

explicit copying;

retention;

promotion;

transfer;

another valid mechanism.



---

100. Lifetime Compatibility

Lifetime compatibility MAY be expressed as:

Provider lifetime >= required lifetime

or through an explicit transformation.

A longer provider lifetime can satisfy a shorter requirement.

A shorter provider lifetime cannot satisfy a longer requirement without extension.


---

101. Lifetime Variance

Lifetime substitution MUST be defined carefully.

A consumer accepting:

Borrowed<T, CallLifetime>

may generally accept a resource guaranteed to live longer than the call.

However, a consumer requiring:

SessionLifetime

cannot safely accept:

CallLifetime

without extension.


---

102. Lifetime Constraints on Generic Types

Generic types MAY carry lifetime constraints.

Conceptually:

Buffer<T, L>

where:

L = lifetime parameter

The implementation may use another internal representation.

ULABI only requires the resulting boundary semantics to remain explicit.


---

103. Lifetime Constraints on Records

A record containing references MUST define the lifetime of each reference.

Example:

Record {
    data: Borrowed<Bytes, CallLifetime>
    owner: Handle<Owner>
}

The record MUST NOT be considered valid after a required field becomes invalid unless the record contract explicitly supports partial validity.


---

104. Partial Invalidity

ULABI SHOULD avoid silently treating partially invalid composite resources as fully valid.

If one mandatory component expires:

Composite
   |
   +-- A valid
   +-- B expired

the composite MUST either:

become invalid;

expose the invalid component explicitly;

support a defined partial-validity state.



---

105. Lifetime and Serialization

Serialization normally creates an independent representation.

Therefore:

serialize(borrowed_resource)

SHOULD produce data that does not depend on the original lifetime after successful serialization.

Deserialization creates a new resource with its own lifetime.

Serialization semantics belong primarily to the distributed/serialization specifications.


---

106. Lifetime and Zero-Copy Serialization

Zero-copy deserialization may preserve dependence on the underlying byte resource.

In that case, the resulting decoded value MUST retain or otherwise depend on the underlying lifetime.

Example:

Byte Buffer
    |
    v
Zero-Copy View

The view MUST NOT outlive the buffer unless it has been materialized or retained independently.


---

107. Lifetime and Security Boundaries

When a resource crosses a sandbox or security boundary, its lifetime MUST remain governed by an explicit contract.

A security boundary MAY revoke access independently of ordinary lifetime expiration.

Therefore:

normal lifetime valid
        |
security revocation
        |
        v
access invalid


---

108. Lifetime and Capability Security

Capabilities MAY encode:

resource identity;

permissions;

lifetime;

expiration;

generation;

revocation state.


A capability MUST NOT grant access beyond the resource's lifetime.

The stronger condition wins:

resource lifetime
        AND
capability lifetime

Effective access exists only while both are valid.


---

109. Effective Lifetime

For a resource accessed through a capability:

EffectiveLifetime =
    ResourceLifetime
    INTERSECT
    CapabilityLifetime
    INTERSECT
    BorrowLifetime

The effective lifetime cannot exceed any mandatory boundary.


---

110. Lifetime Escalation Prohibited

A consumer MUST NOT derive:

longer lifetime

from:

shorter lifetime + copied reference

unless an explicit extension operation authorizes it.


---

111. Lifetime Downgrade

An implementation MAY voluntarily provide a shorter lifetime only when:

1. the interface declares the shorter guarantee;


2. the consumer accepts it;


3. semantic correctness is preserved.



Silent downgrade is prohibited where the lifetime guarantee is normative.


---

112. Lifetime Recovery

A recoverable lifetime may be restored through an explicit recovery operation.

Example:

Expired
   |
recover
   v
New Resource

Recovery MUST NOT magically resurrect an expired object unless the contract explicitly defines resurrection.

Prefer:

Old Resource -> Replacement Resource

rather than:

Old Resource -> mysteriously valid again


---

113. Resurrection

Implementations SHOULD avoid implicit resurrection.

If resurrection is supported, it MUST define:

identity;

ownership;

generation;

capabilities;

lifetime;

state restoration;

failure behavior.



---

114. Lifetime Checkpointing

A resource MAY support checkpoints.

A checkpoint records a recoverable state associated with a lifetime.

Checkpointing MUST NOT extend the live resource lifetime unless explicitly specified.


---

115. Lifetime and Rollback

Rollback may restore logical state without restoring the exact original resource identity.

Therefore:

rollback(resource)

MUST specify whether the resource:

retains identity;

receives a new generation;

is replaced;

is recreated.



---

116. Lifetime and Self-Healing

Self-healing systems MAY repair resources.

The repair mechanism MUST NOT arbitrarily extend resource lifetimes.

A self-healing implementation MUST follow:

Failure
   |
Evidence
   |
Authorized recovery policy?
   |             |
  yes            no
   |             |
Recover        Escalate
   |
Verify
   |
Healthy?
  / \
yes  no
 |    |
Done Rollback/Escalate

Lifetime changes caused by recovery MUST be explicitly recorded in the resource contract.


---

117. Lifetime and Fault Isolation

When a component is isolated after a fault, dependent resources may be revoked.

Isolation MUST NOT silently grant ownership to another component.

Any transfer must be explicitly authorized.


---

118. Lifetime and Quotas

Resource quotas may cause allocations or lifetime extensions to fail.

A quota failure MUST NOT invalidate existing valid resources unless the contract explicitly authorizes reclamation.


---

119. Lifetime and Resource Pressure

Under resource pressure, implementations MAY reclaim resources only according to declared policies.

They MUST NOT silently reclaim resources whose contracts require stronger lifetime guarantees.

If forced reclamation is permitted, the resulting invalidation MUST be observable.


---

120. Lifetime Guarantees

ULABI implementations SHOULD classify lifetime guarantees.

Possible levels:

BestEffort
Bounded
Guaranteed
Persistent

The exact profile semantics must be standardized before these labels become normative.


---

121. Best-Effort Lifetime

A best-effort lifetime means the implementation attempts to preserve the resource but may terminate it under documented failure conditions.

Such a resource MUST NOT be represented as having a stronger guarantee.


---

122. Bounded Lifetime

A bounded lifetime guarantees validity until a specified boundary, subject to declared failure conditions.


---

123. Guaranteed Lifetime

A guaranteed lifetime provides a stronger contract and MUST define exceptional conditions that can still terminate it.


---

124. Persistent Lifetime

A persistent lifetime survives the events explicitly covered by the persistence contract.

Persistence MUST NOT be inferred merely from physical storage survival.


---

125. Conformance Invariants

A conforming implementation MUST satisfy these invariants:

Invariant L1

A resource MUST NOT be accessed before creation.

Invariant L2

A resource MUST NOT be accessed after expiration.

Invariant L3

A resource MUST NOT be accessed after revocation.

Invariant L4

A transferred ownership MUST NOT remain usable by the previous owner.

Invariant L5

A borrowed resource MUST NOT be retained beyond its authorized lifetime.

Invariant L6

A released resource MUST NOT become valid through stale references.

Invariant L7

A stale handle MUST NOT alias a newly allocated unrelated resource.

Invariant L8

Lifetime extension MUST be explicit.

Invariant L9

A lifetime guarantee MUST NOT be silently weakened.

Invariant L10

Asynchronous operations MUST account for resources they retain.

Invariant L11

Dependent resources MUST follow their declared parent lifetime rules.

Invariant L12

Capability validity MUST NOT exceed resource validity.

Invariant L13

Lifetime transitions MUST be deterministic at the contract level.

Invariant L14

Invalid resources MUST produce defined failure behavior.

Invariant L15

Resource identity MUST remain distinguishable across replacement generations.


---

126. Security Requirements

A conforming implementation MUST protect against:

use-after-free;

stale handle reuse;

unauthorized retention;

unauthorized release;

use-after-transfer;

expired capability use;

lifetime confusion;

cross-domain resource confusion;

lifetime extension attacks;

reference substitution;

generation reuse attacks.


Where detection is impossible, the interface MUST prevent the unsafe operation through stronger boundary mechanisms.


---

127. Failure Requirements

Lifetime failures MUST be:

explicit;

deterministic where practical;

distinguishable from successful access;

compatible with the ULABI error model;

safe against resource corruption.


An implementation MUST NOT report an expired resource as valid merely because the underlying bytes still exist physically.


---

128. Compatibility Requirements

Implementations MUST preserve lifetime semantics across compatible ULABI versions.

A version change MUST NOT silently:

shorten a guaranteed lifetime;

extend a lifetime with new side effects;

change ownership;

change borrow duration;

change revocation semantics;

change release semantics.


Such changes require a new incompatible contract or explicit negotiation.


---

129. Versioning

Lifetime contracts MUST be versioned through the applicable ULABI interface/schema versioning mechanism.

Changing:

CallLifetime

to:

SessionLifetime

may be compatible if the stronger guarantee is explicitly permitted.

Changing:

SessionLifetime

to:

CallLifetime

is incompatible unless consumers explicitly accept the weaker contract.


---

130. Feature Negotiation

Optional lifetime features MAY be negotiated.

Examples:

Retention
Promotion
Leases
Revocation
WeakReferences
GenerationProtection
PersistentLifetime

Negotiation MUST NOT silently alter an already established contract.


---

131. Capability Discovery

Implementations MAY advertise support for lifetime capabilities.

Examples:

supports_retention = true
supports_revocation = true
supports_generation_protection = true
supports_persistent_lifetime = false

The discovery mechanism belongs to the ULABI capability-discovery specification.


---

132. Validation

A ULABI validator SHOULD be able to inspect a lifetime-bearing interface and verify:

lifetime mode;

ownership relationship;

retention requirements;

transfer rules;

expiration rules;

revocation rules;

async retention;

dependent resource rules;

capability limits.



---

133. Static Analysis

Language implementations MAY perform compile-time lifetime analysis.

Possible diagnostics include:

borrow may outlive owner
use after lifetime
use after transfer
unauthorized retention
invalid release
stale handle
lifetime mismatch

ULABI does not require compile-time enforcement.


---

134. Dynamic Validation

Runtime implementations MAY enforce lifetimes using:

reference counts;

generation counters;

capability checks;

memory protection;

handle tables;

runtime registries;

leases;

transactions;

sandboxing.


The mechanism is implementation-defined.


---

135. Observability

Lifetime transitions SHOULD be observable through diagnostics where enabled.

Useful events include:

ResourceCreated
LifetimeStarted
BorrowGranted
RetentionGranted
OwnershipTransferred
LifetimeExtended
LifetimeRevoked
LifetimeExpired
ResourceReleased
ResourceReplaced
LifetimeViolation

Observability MUST NOT expose protected data merely to report lifetime events.


---

136. Debugging

Debugging systems SHOULD be able to identify:

resource identity;

generation;

lifetime contract;

owner;

current state;

creation site;

expiration reason;

invalidation reason.


Debug information SHOULD be separable from production semantics.


---

137. Deterministic Testing

Lifetime transitions MUST be testable without requiring nondeterministic garbage-collection timing.

Conformance tests SHOULD explicitly trigger:

expiration;

release;

transfer;

revocation;

retention;

cancellation;

process termination;

session termination;

stale handle use.



---

138. Conformance Test Categories

The ULABI conformance suite SHOULD include:

LIFETIME-001  creation validity
LIFETIME-002  expiration
LIFETIME-003  explicit release
LIFETIME-004  borrow validity
LIFETIME-005  borrow expiration
LIFETIME-006  ownership transfer
LIFETIME-007  transfer invalidation
LIFETIME-008  retention
LIFETIME-009  retention failure
LIFETIME-010  revocation
LIFETIME-011  stale handle protection
LIFETIME-012  generation protection
LIFETIME-013  async retention
LIFETIME-014  async cancellation
LIFETIME-015  view lifetime
LIFETIME-016  nested lifetime
LIFETIME-017  capability lifetime
LIFETIME-018  concurrent release
LIFETIME-019  process termination
LIFETIME-020  distributed expiration
LIFETIME-021  replacement resource
LIFETIME-022  rollback
LIFETIME-023  lifetime negotiation
LIFETIME-024  version compatibility
LIFETIME-025  security violations


---

139. Reference Implementation Requirements

A reference implementation SHOULD provide:

LifetimeManager
LifetimeContract
LifetimeIdentity
LifetimeToken
BorrowManager
RetentionManager
RevocationManager
TransferManager
GenerationTracker
LeaseManager
LifetimeValidator
LifetimeError

These are implementation components, not mandatory programming-language APIs.


---

140. Recommended Internal Architecture

A reference implementation may use:

LifetimeManager
                          |
       +------------------+------------------+
       |                  |                  |
   Contracts          Registry          Validator
       |                  |                  |
       +------------------+------------------+
                          |
                 Resource Identity
                          |
       +------------------+------------------+
       |                  |                  |
    Borrowing         Retention          Revocation
       |                  |                  |
       +------------------+------------------+
                          |
                    State Machine
                          |
                 +--------+--------+
                 |                 |
             Valid              Invalid


---

141. Required Data Structures

A reference implementation SHOULD define equivalents of:

LifetimeContract
LifetimeIdentity
LifetimeState
LifetimeInterval
LifetimePolicy
BorrowContract
RetentionContract
TransferContract
RevocationRecord
ExpirationRecord
LifetimeToken
Generation
ResourceDependency
LifetimeError

The exact language representation is implementation-defined.


---

142. Required Operations

A reference implementation SHOULD expose semantic operations equivalent to:

create_lifetime()
begin_lifetime()
validate_lifetime()
borrow()
retain()
release_retention()
transfer()
revoke()
expire()
release()
renew()
promote()
invalidate()
replace()
check_generation()

Not every operation is required for every ULABI profile.


---

143. Integration With docs/abi/memory-model.md

docs/abi/memory-model.md defines the broad memory boundary.

This document defines the lifetime semantics used by that memory model.

The memory model MUST reference this document for:

lifetime states;

lifetime categories;

borrowing intervals;

invalidation;

transfer;

retention;

expiration.


This document MUST NOT redefine the complete memory-resource representation.


---

144. Integration With docs/memory/allocation.md

docs/memory/allocation.md defines:

allocation;

deallocation;

reallocation;

allocation domains;

allocator identity;

allocation failure.


This document defines:

validity beginning after successful allocation;

lifetime during allocation ownership;

expiration;

release;

transfer;

retention.


Allocation MUST NOT be treated as equivalent to lifetime.


---

145. Integration With docs/memory/ownership.md

ownership.md is authoritative for ownership semantics.

This document consumes ownership concepts to determine lifetime relationships.

Ownership MUST NOT be redefined here.

The ownership document MUST reference this specification for lifetime implications of:

transfer;

borrow;

release;

shared ownership.



---

146. Integration With docs/memory/memory-safety.md

memory-safety.md is authoritative for the broader memory-safety model.

This document provides the lifetime foundation for preventing:

use-after-free;

stale references;

use-after-transfer;

invalid resource access.


The memory-safety specification MUST reference this document rather than duplicating lifetime semantics.


---

147. Integration With docs/abi/calling-convention.md

The calling convention MUST identify parameter and return lifetime semantics where required.

Examples:

Borrowed
Owned
Transferred
Shared
Handle

The calling convention MUST defer detailed lifetime semantics to this document.


---

148. Integration With docs/abi/return-values.md

Return-value contracts MUST identify whether returned resources are:

copied;

borrowed;

transferred;

shared;

handle-based.


The lifetime rules are defined here.


---

149. Integration With docs/interoperability/foreign-function-interface.md

FFI adapters MUST map source-language lifetime mechanisms to ULABI lifetime contracts.

Examples:

C pointer
Rust borrow
Java object reference
Python buffer
Swift object
Go slice

must be translated into explicit ULABI lifetime semantics.

No language-specific lifetime mechanism becomes normative for ULABI.


---

150. Integration With docs/interoperability/cross-language-data.md

Cross-language data contracts MUST include lifetime information whenever data is not independently copied.

Zero-copy and borrowed representations MUST use this specification.


---

151. Integration With docs/interoperability/object-model.md

Object references MUST specify whether they are:

owned;

borrowed;

shared;

weak;

handle-based.


Object lifetime semantics are governed by this document.


---

152. Integration With docs/runtime/runtime-interface.md

Runtime implementations MUST provide whatever mechanisms are necessary to honor lifetime contracts.

The runtime interface MAY expose:

retain;

release;

borrow;

transfer;

revoke;

resource tracking.


The runtime specification MUST NOT redefine the normative lifetime model.


---

153. Integration With docs/runtime/async-model.md

Async operations MUST explicitly account for retained resources.

The async model MUST use this specification for:

operation lifetime;

cancellation;

retention;

completion;

cleanup.



---

154. Integration With docs/runtime/resource-management.md

Resource management MUST use this specification for:

creation;

validity;

expiration;

revocation;

release;

recovery.



---

155. Integration With docs/memory/shared-memory.md

Shared-memory implementations MUST define:

region lifetime;

participant lifetime;

revocation;

destruction;

dependency relationships.


This document provides those lifetime semantics.


---

156. Integration With docs/security/capability-security.md

Capability lifetime MUST NOT exceed resource lifetime.

Capability revocation MUST be able to terminate access according to the security profile.


---

157. Integration With docs/security/security-model.md

Security-sensitive lifetime failures MUST be treated as security-relevant events where appropriate.

The security model governs policy.

This document governs the lifetime mechanics.


---

158. Integration With docs/distributed/serialization.md

Deserialized values have independent lifetimes unless explicitly represented as zero-copy views.

Remote resource lifetime must not be inferred from serialization alone.


---

159. Integration With docs/distributed/remote-calls.md

Remote calls MUST explicitly define:

request lifetime;

response lifetime;

remote borrow;

retention;

cancellation;

timeout;

disconnect behavior.



---

160. Integration With docs/distributed/distributed-abi.md

Distributed ABI contracts MUST distinguish:

local lifetime
remote lifetime
transport lifetime
session lifetime
lease lifetime


---

161. Integration With docs/compatibility/feature-negotiation.md

Lifetime capabilities MAY be negotiated.

Negotiation MUST occur before establishing contracts whose behavior depends on the capability.


---

162. Integration With docs/compatibility/backwards-compatibility.md

Lifetime guarantees are part of interface compatibility.

A change that weakens a guaranteed lifetime MUST be treated as potentially breaking.


---

163. Integration With docs/compatibility/capability-discovery.md

Capability discovery MAY advertise supported lifetime mechanisms.

Examples:

retention
revocation
leases
persistent lifetime
generation protection


---

164. Integration With docs/reliability/self-healing.md

Self-healing MAY create replacement resources.

It MUST NOT silently resurrect invalid resources without an explicit contract.


---

165. Integration With docs/reliability/rollback.md

Rollback MUST preserve defined lifetime invariants.

A rollback operation MUST identify whether it restores or replaces the resource.


---

166. Integration With docs/standards/conformance.md

Conformance claims MUST identify the lifetime capabilities implemented.

Example:

ULABI Memory
    Lifetime: Core
    Borrowing: ✓
    Retention: ✓
    Revocation: ✓
    Generation Safety: ✓
    Async Lifetime: ✓


---

167. Integration With docs/standards/test-suite.md

The conformance test suite MUST include the lifetime tests defined in this document.

Tests MUST cover both:

positive lifetime behavior;

prohibited lifetime behavior.



---

168. Integration With Schemas

The schemas/ directory SHOULD eventually contain machine-readable schemas for:

LifetimeContract
LifetimeIdentity
BorrowContract
RetentionContract
TransferContract
RevocationRecord
LifetimeCapability

Schema identifiers MUST be versioned.


---

169. Integration With Examples

examples/ SHOULD contain examples for:

borrowed buffer
owned buffer
ownership transfer
async retention
zero-copy view
revocation
lease
distributed lifetime
stale handle detection
replacement resource

Examples MUST remain language-neutral where possible.


---

170. Integration With Tests

tests/ SHOULD contain:

lifetime_creation
lifetime_expiration
lifetime_borrow
lifetime_transfer
lifetime_retention
lifetime_revocation
lifetime_generation
lifetime_async
lifetime_concurrency
lifetime_security


---

171. Integration With Conformance

conformance/ SHOULD provide executable tests corresponding to:

LIFETIME-001 ... LIFETIME-025

Implementations should be able to run the suite independently.


---

172. Integration With Reference Implementations

reference/ SHOULD provide at least one implementation demonstrating:

explicit lifetime state;

generation-safe handles;

borrow tracking;

retention;

revocation;

async retention;

deterministic lifetime errors.



---

173. Implementation Independence

No implementation may require:

Zamani;

Sankofa;

C;

Rust;

Java;

Python;

a specific compiler;

a specific operating system;

a specific CPU.


All language implementations are peers.


---

174. Required Conformance Claim

An implementation claiming lifetime conformance MUST identify:

ULABI specification version
Lifetime specification version
supported lifetime categories
supported retention
supported borrowing
supported transfer
supported revocation
supported async semantics
supported distributed semantics
known limitations

"Lifetime compatible" without a capability profile SHOULD NOT be considered sufficient.


---

175. Security Principle

> A lifetime guarantee is a security boundary when access to the resource carries authority.



Therefore lifetime mechanisms MUST NOT be treated merely as memory-management conveniences.

Expiration, revocation, and capability invalidation may be security-critical.


---

176. Design Principle

ULABI lifetime semantics MUST remain:

explicit;

language-neutral;

runtime-neutral;

deterministic at the contract level;

capability-aware;

failure-oriented;

independently implementable;

testable;

versionable;

extensible.



---

177. Final Contract

The essential ULABI lifetime rule is:

> No component may access, retain, transfer, release, or otherwise exercise authority over a ULABI resource beyond the lifetime explicitly granted by its contract.



And:

> A reference does not create a lifetime.



And:

> Lifetime extension must be explicit.



And:

> Expired, revoked, released, or transferred resources must not silently become valid through stale references.




---

178. Completion Criteria

docs/memory/lifetimes.md is complete for this specification phase when:

lifetime states are defined;

lifetime categories are defined;

validity intervals are defined;

borrowing is defined;

retention is defined;

transfer is defined;

revocation is defined;

expiration is defined;

async lifetimes are defined;

view lifetimes are defined;

concurrent lifetime behavior is defined;

capability interaction is defined;

distributed lifetime boundaries are defined;

lifetime errors are defined;

compatibility rules are defined;

conformance requirements are defined;

integration points are defined.


No later document should need to redefine these fundamentals.


---

179. Normative Ownership of Concepts

To prevent specification duplication:

Concept	Authoritative document

Overall architecture	ULABI-DESIGN.md
Core ABI	docs/abi/core-abi.md
ABI memory boundary	docs/abi/memory-model.md
Allocation	docs/memory/allocation.md
Ownership	docs/memory/ownership.md
Lifetimes	docs/memory/lifetimes.md
Memory safety	docs/memory/memory-safety.md
Virtual memory	docs/memory/virtual-memory.md
Shared memory	docs/memory/shared-memory.md
Calling convention	docs/abi/calling-convention.md
Return semantics	docs/abi/return-values.md
Cross-language data	docs/interoperability/cross-language-data.md
FFI	docs/interoperability/foreign-function-interface.md
Object model	docs/interoperability/object-model.md
Async execution	docs/runtime/async-model.md
Resource management	docs/runtime/resource-management.md
Capability security	docs/security/capability-security.md
Distributed calls	docs/distributed/remote-calls.md
Feature negotiation	docs/compatibility/feature-negotiation.md
Compatibility	docs/compatibility/backwards-compatibility.md
Conformance	docs/standards/conformance.md
Tests	docs/standards/test-suite.md


This prevents the lifetime specification from becoming a second memory model.

## What should be done under which files

The important thing is **not to modify all related files now**. The dependency order should be deliberate.

### Phase 1 — independent foundation

These can be completed without waiting for other design documents:

1. `docs/memory/lifetimes.md` — **this document**
   - authoritative lifetime state machine
   - validity intervals
   - expiration
   - borrowing
   - retention
   - transfer
   - revocation
   - async retention
   - lifetime invariants
   - conformance tests
   - all integration contracts predeclared

2. `docs/memory/allocation.md`
   - allocation/deallocation mechanics
   - allocator identity
   - allocation domains
   - size/alignment
   - allocation failure
   - reallocation
   - allocator compatibility
   - must defer lifetime semantics to `lifetimes.md`

3. `docs/memory/ownership.md`
   - authoritative ownership states
   - ownership transfer
   - borrowed/shared ownership
   - ownership authority
   - ownership violations
   - must defer duration/expiration to `lifetimes.md`

4. `docs/memory/memory-safety.md`
   - memory-safety threat model
   - use-after-free prevention
   - stale-reference protection
   - bounds safety
   - race safety
   - must consume lifetime rules rather than redefine them

These four are the **memory contract foundation**.

### Phase 2 — ABI integration

5. `docs/abi/memory-model.md`
   - reference `lifetimes.md`
   - map memory resources to lifetime contracts
   - define boundary representation
   - do not redefine lifetime states

6. `docs/abi/calling-convention.md`
   - parameter ownership/lifetime annotations
   - return ownership/lifetime annotations
   - async call retention
   - ABI-level lifetime metadata

7. `docs/abi/return-values.md`
   - borrowed return
   - owned return
   - transferred return
   - shared return
   - handle return

### Phase 3 — language interoperability

8. `docs/interoperability/language-interoperability.md`
   - map language-specific memory systems to ULABI
   - no language becomes normative

9. `docs/interoperability/foreign-function-interface.md`
   - FFI adapters
   - ownership conversion
   - borrow conversion
   - lifetime validation
   - adapter shims

10. `docs/interoperability/cross-language-data.md`
    - borrowed buffers
    - zero-copy views
    - independent copies
    - lifetime metadata

11. `docs/interoperability/object-model.md`
    - object references
    - ownership
    - weak references
    - shared references
    - object lifetime

### Phase 4 — runtime integration

12. `docs/runtime/runtime-interface.md`
13. `docs/runtime/async-model.md`
14. `docs/runtime/resource-management.md`
15. `docs/runtime/concurrency.md`

These should implement the contracts rather than redefine them.

### Phase 5 — security integration

16. `docs/security/security-model.md`
17. `docs/security/capability-security.md`
18. `docs/security/sandboxing.md`

These establish policy around lifetime; `lifetimes.md` establishes the lifetime mechanics.

### Phase 6 — distributed integration

19. `docs/distributed/remote-calls.md`
20. `docs/distributed/distributed-abi.md`
21. `docs/distributed/serialization.md`
22. `docs/distributed/distributed-errors.md`

Especially important:

```text
local lifetime
      ≠
remote lifetime
      ≠
transport lifetime
      ≠
lease lifetime

Phase 7 — compatibility and standards

23. docs/compatibility/feature-negotiation.md


24. docs/compatibility/backwards-compatibility.md


25. docs/compatibility/capability-discovery.md


26. docs/compatibility/graceful-degradation.md



Then:

27. docs/standards/conformance.md


28. docs/standards/test-suite.md


29. docs/standards/reference-implementations.md


30. docs/standards/certification.md




---

Required code files/modules

I would not put implementation code into docs/. The specification should be language-neutral.

The implementation tree should eventually be structured roughly as:

implementations/
│
├── core/
│   ├── abi/
│   ├── types/
│   ├── interfaces/
│   └── errors/
│
├── memory/
│   ├── allocation/
│   │   ├── allocator.rs
│   │   ├── allocation_domain.rs
│   │   ├── allocation_request.rs
│   │   └── allocation_result.rs
│   │
│   ├── ownership/
│   │   ├── ownership_manager.rs
│   │   ├── ownership_state.rs
│   │   └── transfer.rs
│   │
│   ├── lifetime/
│   │   ├── lifetime_manager.rs
│   │   ├── lifetime_contract.rs
│   │   ├── lifetime_identity.rs
│   │   ├── lifetime_state.rs
│   │   ├── lifetime_interval.rs
│   │   ├── lifetime_policy.rs
│   │   ├── lifetime_token.rs
│   │   ├── borrow_manager.rs
│   │   ├── retention_manager.rs
│   │   ├── revocation_manager.rs
│   │   ├── transfer_manager.rs
│   │   ├── generation_tracker.rs
│   │   ├── expiration_manager.rs
│   │   ├── dependency_tracker.rs
│   │   └── lifetime_error.rs
│   │
│   └── safety/
│       ├── memory_validator.rs
│       ├── stale_handle_detector.rs
│       ├── use_after_free_detector.rs
│       └── bounds_validator.rs
│
├── interoperability/
│   ├── language_adapter.rs
│   ├── ffi_adapter.rs
│   ├── type_mapper.rs
│   ├── lifetime_mapper.rs
│   ├── ownership_mapper.rs
│   ├── object_adapter.rs
│   └── data_bridge.rs
│
├── runtime/
│   ├── resource_manager.rs
│   ├── async_lifetime.rs
│   ├── task_lifetime.rs
│   └── runtime_registry.rs
│
├── security/
│   ├── capability_manager.rs
│   ├── capability_lifetime.rs
│   └── revocation.rs
│
├── distributed/
│   ├── remote_lifetime.rs
│   ├── lease_manager.rs
│   ├── session_lifetime.rs
│   └── remote_resource.rs
│
└── validation/
    ├── contract_validator.rs
    ├── lifetime_validator.rs
    └── compatibility_validator.rs

The .rs suffix above is only an example implementation language. ULABI itself must not require Rust. Equivalent modules could be implemented in C, C++, Rust, Go, Java, Python, Swift, Kotlin, Ada, or another language.

Schemas

Eventually:

schemas/
├── lifetime-contract.schema.json
├── lifetime-identity.schema.json
├── borrow-contract.schema.json
├── retention-contract.schema.json
├── transfer-contract.schema.json
├── revocation-record.schema.json
├── lifetime-capability.schema.json
└── resource-dependency.schema.json

Conformance tests

conformance/
├── memory/
│   ├── lifetime_creation/
│   ├── lifetime_expiration/
│   ├── borrowing/
│   ├── retention/
│   ├── transfer/
│   ├── revocation/
│   ├── generation/
│   ├── async/
│   ├── concurrency/
│   ├── distributed/
│   └── security/
│
└── lifetime/
    ├── LIFETIME-001/
    ├── LIFETIME-002/
    ├── ...
    └── LIFETIME-025/

The key architectural boundary

The final ownership of concepts should be:

allocation.md
      ↓
creates resource

ownership.md
      ↓
defines responsibility

lifetimes.md
      ↓
defines validity duration

memory-safety.md
      ↓
defines safe access

memory-model.md
      ↓
defines how memory crosses ABI boundary

calling-convention.md
      ↓
carries lifetime semantics across calls

FFI
      ↓
maps language mechanisms to ULABI

runtime
      ↓
enforces the contracts

security
      ↓
controls authority

conformance
      ↓
proves compliance

That separation is important because otherwise ULABI will eventually develop several competing definitions of ownership, lifetime, allocation, and memory safety.

