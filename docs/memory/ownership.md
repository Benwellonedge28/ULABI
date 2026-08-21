Yes. I re-read the actual ULABI repository first rather than treating ownership.md as an isolated document.

The repository currently has:

ULABI-DESIGN.md at about 70 KB and explicitly establishes language/runtime/platform/architecture neutrality and implementation independence.

ULABI-SPEC.md already exists.

docs/abi/memory-model.md already defines the boundary memory abstraction, including resource identity, handles and conceptual ownership states.

docs/memory/allocation.md already defines allocation as a separate resource-acquisition/release contract.

docs/memory/lifetimes.md already defines lifetime as a separate authority covering validity, expiration, borrowing intervals, transfer, revocation, asynchronous retention and destruction.

docs/memory/memory-safety.md exists in the repository already, so ownership must integrate with it rather than duplicate it.

docs/memory/ownership.md is currently only a placeholder, making this the correct next foundational document.


So the key architectural decision for this file is:

> ownership.md owns the semantic authority and state-transition rules for ownership. memory-model.md describes ownership as part of a memory resource; lifetimes.md owns temporal validity; allocation.md owns acquisition/release mechanics; memory-safety.md owns enforcement of safety invariants.



That prevents circular redesign later.

docs/memory/ownership.md

Copy the following as the complete file:

# ULABI Ownership Specification

**Universal Language Application Binary Interface**

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Repository:** `Benwellonedge28/ULABI`  
**Document:** `docs/memory/ownership.md`  
**Specification Area:** Memory / Ownership  
**Status:** Architecture / Design Specification  
**Specification Version:** 0.2.0-draft  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI ownership contract.

The purpose of the ownership model is to provide a language-neutral, runtime-neutral, platform-neutral and implementation-independent definition of:

- who has authority over a resource;
- who may transfer that authority;
- who may release a resource;
- who may mutate a resource;
- who may share a resource;
- how ownership changes;
- how ownership interacts with borrowing;
- how ownership interacts with lifetime;
- how ownership interacts with capabilities;
- how ownership interacts with asynchronous execution;
- how ownership interacts with shared memory;
- how ownership failures are represented;
- how independent implementations preserve ownership safety across ULABI boundaries.

ULABI ownership is a **semantic contract**.

It is not a requirement that implementations use:

- Rust-style ownership;
- C++ RAII;
- garbage collection;
- reference counting;
- manual memory management;
- affine types;
- linear types;
- borrow checking;
- unique pointers;
- shared pointers;
- any particular compiler;
- any particular runtime.

The implementation mechanism is implementation-defined.

The ownership behavior visible across a ULABI boundary is normative.

---

# 2. Relationship to the ULABI Architecture

`ULABI-DESIGN.md` establishes ULABI as a universal interoperability contract rather than a programming language, compiler, runtime, operating system, processor architecture or memory-management system.

The architecture requires:

- language independence;
- runtime independence;
- platform independence;
- architecture independence;
- explicit ownership boundaries;
- explicit lifetime boundaries;
- safe memory interoperability;
- capability-based security;
- process isolation;
- deterministic behavior;
- zero-copy interoperability where safe;
- implementation independence.

This document specializes those principles for ownership.

The relationship is:

```text
                         ULABI
                           |
                    Memory Contract
                           |
                    Ownership Contract
                           |
        +------------------+------------------+
        |                  |                  |
     Authority          Transfer          Sharing
        |                  |                  |
        +------------------+------------------+
                           |
                       Lifetime
                           |
                       Safety
                           |
                    Implementation

Ownership MUST remain separate from:

allocation;

lifetime;

memory safety;

virtual memory;

shared-memory synchronization;

capability authorization;

language-specific memory models.



---

3. Non-Goals

This specification does not:

1. Define a universal programming language.


2. Define a universal garbage collector.


3. Define a universal allocator.


4. Require manual memory management.


5. Require garbage collection.


6. Require reference counting.


7. Require a particular ownership type system.


8. Require a particular pointer representation.


9. Require all implementations to expose pointers.


10. Replace language-specific ownership systems.


11. Define virtual-memory behavior.


12. Define physical-memory management.


13. Define synchronization algorithms.


14. Define cryptographic authorization mechanisms.


15. Define a particular operating-system resource manager.


16. Make remote ownership identical to local ownership.


17. Make ownership equivalent to capability authorization.


18. Make ownership equivalent to lifetime.


19. Make copying automatically transfer ownership.


20. Make references automatically become owners.




---

4. Fundamental Principle

ULABI separates:

Language Ownership Model
          |
          v
   ULABI Ownership Contract
          |
          v
Implementation Ownership Model

Examples of implementation models include:

Rust ownership
C++ RAII
C manual ownership
C++ smart pointers
Java garbage collection
Python reference management
Swift ARC
Go garbage collection
Zamani ownership model
Sankofa ownership model

These systems may independently implement the ULABI ownership contract.

ULABI MUST NOT merge them.

ULABI MUST NOT require one language to adopt another language's ownership model.


---

5. Definition of Ownership

For ULABI purposes:

> Ownership is the authoritative semantic authority to determine the disposition of a resource under its ownership contract.



Ownership may include authority to:

retain;

transfer;

release;

mutate;

share;

delegate;

revoke subordinate access;

determine final disposition.


The exact authorities MUST be defined by the resource contract.

Ownership does not automatically imply every possible permission.

For example:

Owner
  |
  +-- may release
  +-- may transfer
  +-- may mutate

is valid only when those rights are included in the ownership contract.


---

6. Ownership Is Not Authorization

Ownership and capability authorization are distinct.

Conceptually:

Ownership
    |
    | determines disposition authority
    v
Resource

Capability
    |
    | determines permitted operation
    v
Resource

An owner may delegate limited access.

A non-owner may receive authorized access.

Therefore:

Owner != only possible user

and:

User != necessarily owner

The capability model is defined by:

docs/security/capability-security.md.


---

7. Ownership Is Not Lifetime

Ownership and lifetime are related but distinct.

Ownership answers:

> Who has authoritative disposition authority?



Lifetime answers:

> During what interval is the resource valid?



A resource may remain valid while ownership changes.

Example:

Owner A
   |
 transfer
   v
Owner B
   |
   v
same resource

The resource remains valid across the ownership transition if the transfer succeeds.

Lifetime semantics are defined by:

docs/memory/lifetimes.md.


---

8. Ownership Is Not Allocation

Allocation creates or acquires a resource.

Ownership determines authority over that resource.

Conceptually:

Allocation
    |
    v
Resource Created
    |
    v
Ownership Established

Allocation semantics are defined by:

docs/memory/allocation.md.

Ownership MUST NOT redefine allocator behavior.


---

9. Ownership States

ULABI defines the following conceptual ownership states:

Unowned
Owned
Borrowed
Shared
ImmutableShared
Transferred
Released
Revoked
Invalid

Not every resource needs to expose every state.

The states describe semantic behavior, not mandatory internal data structures.


---

10. Unowned State

A resource is Unowned when no component has authoritative ownership under the resource contract.

This state MAY occur:

before ownership assignment;

during controlled initialization;

for globally immutable resources;

for resources governed by another authority.


An unowned resource MUST NOT be treated as freely mutable.


---

11. Owned State

An Owned resource has one authoritative owner.

Conceptually:

Owner A
   |
   v
Resource R

The owner is the authority responsible for the resource's disposition under the ownership contract.

Unless explicitly delegated, an exclusive owner has authority to:

transfer ownership;

release the resource;

grant borrowing;

grant sharing;

perform owner-authorized mutation.



---

12. Single-Owner Invariant

For resources requiring exclusive ownership:

> At most one authoritative exclusive owner may exist at any time.



Invalid:

Owner A ----+
            |
            v
         Resource R
            ^
            |
Owner B ----+

Valid:

Owner A
   |
   v
Resource R

followed by:

Owner A
   |
 transfer
   v
Owner B

A successful transfer MUST invalidate the previous exclusive ownership authority.


---

13. Ownership Identity

An ownership authority MUST be semantically identifiable.

The identity MUST NOT depend solely on:

a raw pointer;

a virtual address;

a physical address;

an allocator address;

an object address;

a process-local address.


Possible implementation mechanisms include:

capability identifiers;

protected handles;

runtime ownership tables;

generation counters;

unique resource identifiers;

ownership tokens.


The exact mechanism is implementation-defined.


---

14. Ownership Record

A conceptual ownership record MAY contain:

OwnershipRecord {
    resource_id
    owner_id
    ownership_kind
    generation
    permissions
    lifetime_reference
    transfer_state
    sharing_state
}

This is a semantic model.

Implementations MAY use a different representation.


---

15. Ownership Kinds

ULABI SHOULD distinguish:

Exclusive
Shared
ImmutableShared
Borrowed
Delegated
Transferred
Revoked

The exact representation is implementation-defined.


---

16. Exclusive Ownership

Exclusive ownership grants one authoritative owner.

Exclusive ownership is appropriate for resources where concurrent independent disposition or mutation would be unsafe.

Examples include:

mutable buffers;

exclusive device resources;

allocator-managed resources;

unique operating-system handles.



---

17. Shared Ownership

Shared ownership allows multiple participants to retain ownership authority according to a defined shared-ownership contract.

Shared ownership MUST define:

how ownership is acquired;

how ownership is relinquished;

when the resource becomes releasable;

who may mutate;

whether mutation is synchronized;

how lifetime is maintained;

how final destruction occurs.


Shared ownership MUST NOT be inferred merely from the existence of multiple references.


---

18. Immutable Shared Ownership

Immutable shared ownership permits multiple consumers to retain access while preventing unauthorized mutation.

Conceptually:

Immutable Resource
              /       |       \
             /        |        \
        Consumer A Consumer B Consumer C

This is the preferred ownership model for many safe zero-copy operations.


---

19. Borrowing

Borrowing grants temporary access without transferring ownership.

Conceptually:

Owner A
   |
   +---- Borrow ----> Consumer B

Consumer B does not become the owner.

The borrow MUST have an explicit or inferable validity interval.

Borrowing semantics integrate with:

docs/memory/lifetimes.md.


---

20. Borrowing Does Not Transfer Ownership

The following is invalid:

Owner A
   |
 borrow
   v
Consumer B
   |
   +-- releases resource

unless the resource contract explicitly grants release authority.

A borrowed reference MUST NOT automatically gain ownership authority.


---

21. Borrowed Mutation

A borrowed resource MAY be mutable only when the ownership contract explicitly grants mutation authority.

For example:

Owner
  |
  +-- exclusive mutable borrow --> Consumer

may be valid.

However:

Owner
  |
  +-- read-only borrow --> Consumer

MUST NOT permit mutation.


---

22. Borrowing and Exclusive Ownership

A resource with exclusive ownership MAY temporarily grant an exclusive mutable borrow.

During that borrow:

Owner
  |
  X conflicting mutation
  |
Borrower

The owner MUST NOT exercise conflicting access.

The implementation MUST prevent or detect conflicting ownership authority.


---

23. Shared Read Borrowing

Multiple read-only borrows MAY coexist where the resource is immutable or where the ownership contract permits concurrent reads.

Resource
 /   |   \
R1   R2   R3

No reader gains mutation or release authority merely by reading.


---

24. Mutable Aliasing

Multiple independent mutable aliases MUST NOT exist for an exclusively mutable resource unless the resource contract explicitly defines synchronization and shared mutation semantics.

This prevents ownership ambiguity and data races.


---

25. Ownership Transfer

Ownership transfer is an atomic semantic transition:

Owner A
   |
   | transfer
   v
Owner B

A successful transfer MUST establish:

Owner B = authoritative owner
Owner A = no longer authoritative owner

The resource itself remains valid unless the transfer contract states otherwise.


---

26. Transfer Preconditions

A transfer MUST establish:

1. source authority is valid;


2. destination identity is valid;


3. resource is transferable;


4. resource is not already released;


5. resource is not revoked;


6. resource lifetime permits transfer;


7. required capabilities exist;


8. no conflicting borrow prevents transfer;


9. required synchronization conditions hold.



If any mandatory condition fails, transfer MUST fail without creating two owners.


---

27. Atomic Transfer

Ownership transfer MUST be atomic from the perspective of observers.

The system MUST NOT expose:

Owner A + Owner B

as simultaneously authoritative exclusive owners.

A transfer MAY internally involve multiple steps, but the externally observable ownership state MUST remain coherent.


---

28. Failed Transfer

A failed transfer MUST leave the source ownership state valid unless the contract explicitly defines another atomic state transition.

Example:

Owner A
   |
 failed transfer
   X
   |
Owner A remains owner

The implementation MUST NOT silently discard ownership.


---

29. Transfer Across Processes

Cross-process transfer MUST use an explicit mechanism.

A raw pointer MUST NOT be treated as transferable ownership.

Possible mechanisms include:

protected handles;

operating-system resource transfer;

shared-memory descriptors;

capability transfer;

serialized resource descriptors.


The mechanism is implementation-defined.


---

30. Transfer Across Machines

Distributed ownership transfer MUST account for:

identity;

authorization;

failure;

replay;

duplication;

timeout;

disconnection;

confirmation;

rollback.


A network message claiming ownership MUST NOT automatically establish ownership.

Distributed semantics are defined by:

docs/distributed/remote-calls.md

and related distributed specifications.


---

31. Ownership Transfer and Serialization

Serialization of a value does not automatically transfer ownership of the original resource.

For example:

serialize(R)

creates a representation of data.

It does not automatically mean:

ownership(R) -> receiver

Explicit transfer semantics are required.


---

32. Ownership Delegation

An owner MAY delegate specific authority without transferring ownership.

Example:

Owner A
   |
   +-- read capability --> B
   |
   +-- mutate capability --> C

A remains owner.

Delegation MUST define:

permitted operations;

scope;

duration;

revocation;

whether further delegation is allowed.



---

33. Delegated Authority

Delegated authority MUST NOT exceed the authority available to the delegating owner.

A component MUST NOT delegate rights it does not possess.


---

34. Ownership Revocation

An ownership or delegated-authority revocation MUST prevent further exercise of the revoked authority according to the resource contract.

Revocation MUST NOT silently create a second owner.

Where ownership itself cannot be revoked safely, the implementation MUST use the resource's defined lifecycle transition, such as:

revoke access
isolate resource
destroy resource
transfer authority
escalate failure


---

35. Ownership Generation

Implementations SHOULD associate ownership authority with a generation or equivalent freshness mechanism.

Example:

Resource R
Owner A
Generation 7

After transfer:

Resource R
Owner B
Generation 8

An old ownership token for generation 7 MUST NOT authorize generation-8 ownership operations.


---

36. Stale Ownership

A stale ownership reference MUST NOT regain authority.

This includes:

old handles;

copied ownership tokens;

cached owner records;

stale wrappers;

expired delegates;

old process references.



---

37. Ownership and Lifetime

Ownership transitions MUST remain consistent with the lifetime state.

Examples:

Valid + Owned       -> valid
Valid + Transferred -> valid
Expired + Owned     -> invalid
Released + Owned    -> invalid
Revoked + Owned     -> contract-defined invalid/restricted state

Ownership MUST NOT resurrect an expired resource.

Ownership transfer MUST NOT automatically extend a lifetime unless explicitly defined.


---

38. Ownership and Destruction

Final destruction occurs only when the resource's ownership and lifetime contracts permit destruction.

For exclusive ownership:

Owner
  |
release
  |
  v
Destroyed/Released

For shared ownership:

Owner A ----+
Owner B ----+----> final owner released
Owner C ----+
                  |
                  v
              destruction

The exact destruction mechanism is implementation-defined.


---

39. Release Authority

Only an authority explicitly permitted to release a resource may release it.

A borrowed read-only reference MUST NOT automatically possess release authority.

A delegated release capability MUST be explicit.


---

40. Double Release

A resource MUST NOT successfully transition from:

Released

to:

Released again

under the same ownership contract.

A second release MUST either:

fail;

be explicitly idempotent;

or be handled by a defined resource-specific policy.


It MUST NOT corrupt unrelated resources.


---

41. Ownership and Allocation

Allocation establishes a resource according to:

docs/memory/allocation.md.

The allocation contract MUST identify the initial ownership state.

Possible states include:

Allocated -> Owned
Allocated -> Shared
Allocated -> Unowned

The exact transition MUST be defined by the allocator/resource contract.


---

42. Ownership and Memory Safety

Ownership is one of the foundations of memory safety.

docs/memory/memory-safety.md MUST enforce the following ownership invariants:

no unauthorized owner;

no double exclusive ownership;

no use after ownership transfer;

no release without authority;

no mutation without authority;

no ownership through stale references;

no ownership resurrection after invalidation.


Memory-safety enforcement remains the responsibility of memory-safety.md.

This document defines the ownership semantics that it enforces.


---

43. Ownership and Type Safety

Ownership authority MUST be associated with the correct resource identity and type.

A valid owner of:

Resource<T>

MUST NOT automatically become an owner of:

Resource<U>

where T and U are incompatible.

Type compatibility is defined elsewhere in ULABI.


---

44. Ownership and Handles

Opaque handles MAY represent ownership authority.

A handle MUST NOT automatically imply unrestricted ownership.

Conceptually:

Handle<R>
   |
   +-- identity
   +-- generation
   +-- authority
   +-- permissions

The handle's authority is determined by its contract.


---

45. Ownership and Pointers

A raw pointer MUST NOT automatically establish ULABI ownership.

A pointer may represent:

an address;

a borrowed reference;

an internal implementation detail;

an unsafe reference;

an explicitly defined ULABI resource.


Only an explicit contract can establish ownership.


---

46. Ownership and Views

A view generally does not own its underlying resource.

Conceptually:

Owner
  |
  v
Resource
  |
  v
View

The view MUST NOT outlive the resource unless it acquires an explicit lifetime-retaining ownership/reference authority.

View semantics integrate with:

docs/abi/memory-model.md

and:

docs/memory/lifetimes.md.


---

47. Ownership and Zero-Copy

Zero-copy sharing MUST establish an ownership/lifetime contract.

A zero-copy consumer MUST know whether it:

borrows;

shares ownership;

receives ownership;

receives immutable access;

receives delegated access.


Zero-copy MUST NOT create implicit ownership.


---

48. Ownership and Asynchronous Execution

An asynchronous operation MUST retain or receive an appropriate ownership/reference authority for every resource it uses after the initiating call returns.

Invalid:

submit(buffer)
return
release(buffer)
worker -> uses buffer

Valid alternatives include:

submit(buffer)
   |
   +-- retain/borrow with valid lifetime
   |
   v
worker

or:

submit(buffer)
   |
   +-- ownership transfer
   |
   v
worker

The asynchronous contract MUST identify which model applies.


---

49. Ownership and Cancellation

Cancellation MUST define what happens to ownership held by the cancelled operation.

Cancellation MUST NOT cause:

double release;

ownership loss;

ownership duplication;

use after release;

stale ownership;

permanent ownership deadlock.



---

50. Ownership and Concurrency

Ownership authority MUST remain well-defined under concurrent execution.

An implementation MUST NOT create multiple exclusive owners merely because multiple threads execute concurrently.

Concurrent shared ownership MUST define:

synchronization;

mutation authority;

release semantics;

lifetime;

visibility;

ordering.


Concurrency semantics are defined by the runtime and shared-memory specifications.


---

51. Ownership and Shared Memory

Shared-memory resources MUST explicitly distinguish:

shared ownership

from:

shared access

Multiple processes may access a resource without all becoming owners.

Ownership remains an explicit authority.

Shared-memory synchronization is defined by:

docs/memory/shared-memory.md.


---

52. Ownership and Virtual Memory

Virtual addresses do not establish ownership.

Mapping a resource into an address space does not automatically transfer ownership.

Conceptually:

Resource
   |
   +-- mapped into Process A
   |
   +-- mapped into Process B

does not imply:

Process A owns
Process B owns

unless the ownership contract explicitly says so.

Virtual-memory semantics are defined by:

docs/memory/virtual-memory.md.


---

53. Ownership and Security

Ownership MUST NOT bypass security policy.

A component may be the owner of a resource while still being subject to:

sandbox policy;

process isolation;

security policy;

platform restrictions;

hardware restrictions.


Conversely, a component may receive authorized access without ownership.


---

54. Ownership and Capability Security

Capability-based security SHOULD represent ownership authority as a capability or capability-associated state where appropriate.

However:

capability != ownership

unless the resource contract explicitly defines the capability as ownership authority.

Capability semantics are authoritative in:

docs/security/capability-security.md.


---

55. Ownership Across Trust Boundaries

Ownership transfer across trust boundaries MUST require explicit authorization.

Trust boundaries include:

process boundaries;

sandbox boundaries;

container boundaries;

virtual-machine boundaries;

machine boundaries;

network boundaries;

security-domain boundaries.


An ownership claim from an untrusted component MUST NOT be accepted without verification.


---

56. Ownership Proof

Where a high-assurance profile requires proof of ownership, the implementation MAY use:

signed ownership tokens;

protected handles;

cryptographic capabilities;

trusted runtime state;

hardware-backed authority;

formally verified state transitions.


The mechanism is implementation-defined.


---

57. Ownership State Machine

The conceptual ownership state machine is:

+-----------+
                     |  Unowned  |
                     +-----------+
                           |
                      acquire/assign
                           |
                           v
                     +-----------+
                     |   Owned   |
                     +-----------+
                       /   |   \
                      /    |    \
                   borrow  |   share
                    /      |      \
                   v       |       v
              +---------+  |  +---------+
              | Borrowed|  |  | Shared  |
              +---------+  |  +---------+
                   |       |       |
                   |       |       |
                return     |    release
                   |       |       |
                   +-------+-------+
                           |
                       transfer
                           |
                           v
                     +-----------+
                     | Transferred|
                     +-----------+
                           |
                           v
                         Owned
                           |
                        release
                           |
                           v
                     +-----------+
                     | Released  |
                     +-----------+
                           |
                           v
                       Invalid

Revocation may occur from any applicable authorized state:

Valid ownership
      |
    revoke
      v
Revoked
      |
      +--> release
      +--> isolate
      +--> transfer
      +--> destroy
      +--> recover

The exact transition is resource-specific.


---

58. Valid Ownership State

A resource has valid ownership only if all applicable conditions hold:

ValidOwnership =
    ResourceExists
    AND OwnerIdentityValid
    AND OwnershipGenerationValid
    AND OwnershipNotRevoked
    AND LifetimeAllowsOwnership
    AND CapabilityAllowsOperation

The implementation MAY represent these conditions differently.

The semantic result is normative.


---

59. Ownership Invariants

The following invariants are normative.

Invariant 1 — Single Exclusive Owner

An exclusively owned resource MUST have at most one authoritative owner.

Invariant 2 — No Stale Authority

Expired or stale ownership references MUST NOT regain authority.

Invariant 3 — Transfer Atomicity

Successful transfer MUST produce exactly one authoritative destination owner.

Invariant 4 — Failed Transfer Preservation

A failed transfer MUST NOT silently destroy the source's valid ownership.

Invariant 5 — No Unauthorized Release

Only an authorized owner/delegate may release the resource.

Invariant 6 — No Unauthorized Mutation

Ownership or delegated mutation authority MUST be required for mutable access.

Invariant 7 — No Ownership Resurrection

Released or invalid resources MUST NOT become owned again merely because an old reference exists.

Invariant 8 — Lifetime Consistency

Ownership MUST NOT override lifetime expiration.

Invariant 9 — Capability Consistency

Ownership MUST NOT bypass capability restrictions.

Invariant 10 — Type Consistency

Ownership authority MUST remain associated with the correct resource identity and type.

Invariant 11 — No Implicit Ownership

Borrowing, viewing, copying, serialization and mapping MUST NOT implicitly transfer ownership.

Invariant 12 — Concurrency Consistency

Concurrent execution MUST NOT silently create conflicting exclusive ownership.


---

60. Ownership Operations

A ULABI implementation SHOULD provide semantic operations equivalent to:

acquire(resource)
assign(resource, owner)
borrow(resource, borrower, contract)
share(resource, participant, contract)
delegate(resource, authority, contract)
transfer(resource, destination)
retain(resource)
release(resource)
revoke(resource, authority)
query_owner(resource)
query_ownership_state(resource)

These names are conceptual.

A language binding MAY use different names.


---

61. Acquire

acquire establishes ownership authority according to the resource contract.

The operation MUST define:

resource state before acquisition;

resource state after acquisition;

acquiring identity;

required capability;

failure behavior.



---

62. Assign

assign establishes an authoritative owner for an unowned or newly created resource.

Assignment MUST NOT create multiple authoritative exclusive owners.


---

63. Borrow

borrow creates non-owning access.

A borrow MUST define:

borrower;

access mode;

scope;

lifetime;

mutation authority;

revocation;

return/end condition.



---

64. Share

share establishes shared ownership or shared authority where permitted.

The sharing contract MUST define:

participant admission;

participant removal;

mutation semantics;

release semantics;

final ownership;

lifetime.



---

65. Delegate

delegate grants limited authority without transferring ownership.

Delegation MUST define:

scope;

operations;

duration;

revocation;

further delegation policy.



---

66. Transfer

transfer moves ownership authority from one owner to another.

The operation MUST be atomic at the semantic boundary.


---

67. Retain

retain extends or establishes a valid reference/ownership relationship where the resource contract supports retention.

Retention MUST NOT violate the resource's lifetime or ownership rules.


---

68. Release

release relinquishes ownership authority.

Release semantics are coordinated with:

docs/memory/allocation.md

and:

docs/memory/lifetimes.md.

Release MUST NOT automatically imply physical memory deallocation in every implementation.


---

69. Revoke

revoke invalidates previously granted ownership-related authority where revocation is supported.

Revocation MUST be explicit and auditable where required.


---

70. Ownership Errors

Ownership-related failures SHOULD use stable semantic error classes.

Recommended errors include:

OwnershipDenied
OwnershipConflict
InvalidOwner
StaleOwnership
OwnershipRevoked
OwnershipExpired
OwnershipTransferFailed
OwnershipTransferConflict
OwnershipAlreadyTransferred
InvalidBorrow
BorrowConflict
InvalidDelegation
DelegationRevoked
UnauthorizedRelease
UnauthorizedMutation
AlreadyReleased
InvalidOwnershipState
OwnershipTypeMismatch
OwnershipCapabilityDenied

The final error encoding MUST integrate with:

docs/abi/exception-model.md.


---

71. Failure Atomicity

Ownership operations SHOULD be failure-atomic.

For example:

transfer(A -> B)

MUST result in either:

A remains owner

or:

B becomes owner

and MUST NOT expose an ambiguous state where neither or both are authoritative.


---

72. Recovery from Ownership Failure

Ownership failures MUST be handled according to explicit policy.

A failure MUST NOT cause arbitrary ownership reassignment.

Recovery may include:

detect
  |
isolate
  |
verify state
  |
restore valid state
  |
continue

or:

detect
  |
isolate
  |
escalate

Recovery semantics integrate with:

docs/reliability/fault-detection.md;

docs/reliability/fault-isolation.md;

docs/reliability/recovery.md;

docs/reliability/rollback.md;

docs/reliability/self-healing.md.



---

73. No Autonomous Ownership Mutation

ULABI MUST NOT permit an implementation to arbitrarily modify ownership merely because:

a failure occurred;

a timeout occurred;

a component requested it;

an optimization is possible;

a self-healing mechanism is active.


Ownership mutation MUST follow an explicit authorized policy.


---

74. Ownership and Self-Healing

Self-healing mechanisms MAY recover ownership state only when a defined recovery policy permits it.

For example:

Ownership failure
       |
       v
Evidence
       |
       v
Known recovery policy?
      / \
    YES  NO
     |    |
 recover escalate
     |
 verify
     |
 valid?
   / \
 YES  NO
 |     |
done rollback/escalate

Self-healing MUST NOT bypass ownership invariants.


---

75. Ownership and Rollback

If a transfer or ownership-changing operation is transactional, rollback MUST restore a previously valid ownership state.

Rollback MUST NOT create:

duplicate owners;

orphaned resources;

stale authority;

invalid lifetime state.



---

76. Ownership and Distributed Failure

Distributed ownership transitions MUST account for partial failure.

The implementation MUST define behavior for:

sender failure;

receiver failure;

network partition;

duplicate message;

replay;

timeout;

acknowledgement loss;

stale ownership information.


No component may assume that sending an ownership message alone proves successful transfer.


---

77. Ownership and Idempotency

Where distributed ownership operations may be retried, operations SHOULD be idempotent or use unique transaction identifiers.

Example:

transfer_id
resource_id
source_generation
destination_id

This prevents duplicate transfers from creating inconsistent ownership state.


---

78. Ownership and Replay Protection

Ownership-changing commands SHOULD contain sufficient freshness information to prevent replay.

Possible mechanisms include:

generations;

sequence numbers;

nonces;

transaction identifiers;

capability epochs.



---

79. Ownership and Versioning

Ownership semantics MUST remain stable across compatible ULABI versions.

A new version MUST NOT silently change:

who is considered owner;

what transfer means;

what release means;

what borrowing means;

what stale ownership means.


Breaking ownership semantics require an explicit compatibility boundary.

Integration:

ULABI-VERSIONING.md;

docs/compatibility/backwards-compatibility.md;

docs/compatibility/forwards-compatibility.md;

docs/compatibility/feature-negotiation.md.



---

80. Feature Negotiation

Optional ownership capabilities MAY be negotiated.

Examples:

ExclusiveOwnership
SharedOwnership
Borrowing
Delegation
Revocation
DistributedTransfer
HardwareOwnership
PersistentOwnership

A component MUST NOT assume an optional ownership feature exists unless:

it is mandatory for the negotiated profile; or

capability discovery confirms support.



---

81. Compatibility

A ULABI implementation claiming ownership compatibility MUST declare which ownership semantics it supports.

For example:

ULABI Core
Ownership
  Exclusive       ✓
  Borrowing       ✓
  Shared          ✓
  Delegation      ✓
  Revocation      ✓
  Distributed     —

An implementation MUST NOT claim support for a semantic feature it cannot enforce.


---

82. Language Mapping

Each language binding SHOULD document how its native ownership model maps to ULABI.

Examples:

Language Model
      |
      v
ULABI Adapter
      |
      v
ULABI Ownership Contract

The mapping document MUST identify:

native owner representation;

borrow representation;

transfer representation;

release representation;

lifetime interaction;

unsupported semantics;

unsafe escape hatches.


The ULABI core specification remains independent of the language.


---

83. Garbage-Collected Languages

Garbage-collected implementations MAY map ownership to runtime-managed objects.

However, garbage collection MUST NOT be interpreted as proof that:

ownership is transferable;

a resource is immutable;

a resource is authorized;

a resource remains valid indefinitely.


External resources MUST still follow explicit ownership contracts.


---

84. Reference-Counted Languages

Reference counting MAY implement shared ownership.

Reference counting alone MUST NOT be considered sufficient proof of:

exclusive ownership;

capability authority;

absence of cycles;

race freedom;

lifetime correctness.



---

85. Manual-Memory Languages

Manual-memory implementations MAY map:

allocate -> acquire
free     -> release

but MUST additionally enforce ULABI ownership semantics.

A manual free operation MUST NOT bypass ownership authorization.


---

86. Ownership-Based Languages

Languages with static ownership systems MAY map their native ownership states directly to ULABI.

However, the ULABI boundary MUST expose semantic ownership behavior rather than requiring consumers to understand source-language syntax.


---

87. Unsafe Languages

Unsafe languages MAY implement ULABI.

Unsafe internal operations MUST remain internal unless they satisfy the ULABI ownership contract.

An unsafe operation crossing the boundary MUST be explicitly classified and MUST NOT be represented as a safe ownership operation without verification.


---

88. FFI Ownership Contract

FFI boundaries MUST explicitly define:

who owns input
who owns output
who may mutate
who may release
whether ownership transfers
whether the resource is borrowed
how long it remains valid

The FFI MUST NOT infer ownership solely from native function signatures when the ownership contract is ambiguous.

Integration:

docs/interoperability/foreign-function-interface.md.


---

89. Callback Ownership

Callbacks MUST define ownership of:

callback objects;

callback context;

arguments;

returned resources;

retained state.


A callback MUST NOT retain a borrowed resource beyond its authorized lifetime.


---

90. Ownership and Exceptions

An exception or error MUST NOT silently alter ownership unless the operation's contract explicitly defines that behavior.

For example:

operation(resource)
throws Error

MUST define whether the resource remains:

owned by caller
owned by callee
transferred
released
unchanged

This is essential for cross-language interoperability.


---

91. Ownership and Return Values

Returned resources MUST have explicit ownership semantics.

A function returning:

Resource<T>

MUST specify whether the caller receives:

ownership;

borrowed access;

shared ownership;

immutable shared access;

a capability;

a view.


The calling convention and return-value specifications integrate with:

docs/abi/calling-convention.md;

docs/abi/return-values.md.



---

92. Ownership and Parameters

Parameters crossing a ULABI boundary MUST declare ownership semantics.

Conceptually:

borrow(input)
move(input)
share(input)
copy(input)
delegate(input)

The exact syntax is language-specific.

The semantic mode MUST be discoverable from the interface contract.


---

93. Ownership and Generic Types

Generic resources MUST preserve ownership semantics independently of the concrete type.

For:

Container<T>

the ownership contract MUST specify whether ownership applies to:

the container;

its elements;

both;

neither;

conditionally depending on T.



---

94. Ownership and Composite Values

A composite value may contain multiple owned resources.

The ownership contract MUST define whether ownership is:

shallow;

deep;

borrowed;

shared;

transferred recursively.


Implicit assumptions MUST NOT be made.


---

95. Ownership and Copying

Copying a value MUST NOT automatically imply copying ownership.

The contract MUST distinguish:

copy data

from:

copy ownership authority

Exclusive ownership authority MUST NOT be duplicated through an ordinary copy.


---

96. Ownership and Cloning

A clone operation MAY create a new independently owned resource.

Conceptually:

Resource A
    |
   clone
    |
    +----> Resource B

The new resource has independent identity and ownership.

Cloning MUST NOT create two owners of the same exclusive resource.


---

97. Ownership and Serialization

Serialization creates a representation.

Deserialization creates or acquires a new resource unless an explicit resource-transfer protocol is defined.

Therefore:

serialize(R)

does not automatically preserve ownership of R.


---

98. Ownership and Memory Mapping

Mapping a resource into a process does not automatically transfer ownership.

If mapping grants ownership, that transfer MUST be explicit.

Otherwise the mapping represents a borrow, shared access or delegated authority.


---

99. Ownership and Persistence

Persistent storage MUST NOT preserve ownership authority merely by preserving bytes.

After restart, ownership MUST be re-established through an explicit recovery or restoration protocol.


---

100. Ownership and Device Resources

Device and accelerator resources MUST define ownership independently of host-language references.

Examples:

GPU buffers;

NPU tensors;

FPGA regions;

DMA buffers;

device queues.


Device ownership MUST account for:

device access;

host access;

synchronization;

lifetime;

transfer;

revocation.



---

101. Ownership and Hardware

Hardware-backed ownership MAY use:

memory protection;

IOMMU permissions;

protected memory;

hardware capabilities;

device isolation.


Hardware mechanisms are implementation-specific.

The semantic ownership contract remains ULABI-defined.


---

102. Ownership and Real-Time Systems

Ownership operations used in real-time profiles MUST define their timing characteristics where required.

Implementations SHOULD avoid unbounded ownership operations in hard real-time contexts.

If an ownership operation cannot meet the profile's timing requirement, the implementation MUST report the appropriate capability or profile limitation.


---

103. Ownership and Embedded Systems

Ownership semantics MUST remain implementable on constrained systems.

A conforming embedded implementation does not need:

garbage collection;

dynamic allocation;

heavyweight runtime metadata;


unless required by a selected profile.

The semantic ownership contract remains the same.


---

104. Ownership and Distributed Systems

Distributed ownership MUST distinguish:

local authority

from:

remote claim

A remote claim becomes authoritative only after the applicable distributed protocol establishes it.


---

105. Ownership Consistency

For resources with a single global owner, the distributed implementation MUST establish a single authoritative ownership state.

If strong global ownership cannot be guaranteed, the resource MUST use an explicitly weaker ownership model.

The implementation MUST NOT falsely claim exclusive global ownership.


---

106. Ownership Conflict

When conflicting ownership states are detected:

Owner A
   |
   v
Resource R
   ^
   |
Owner B

the implementation MUST NOT silently choose an owner unless an explicit conflict-resolution policy exists.

Possible responses:

reject;

isolate;

reconcile;

rollback;

invoke consensus;

escalate.



---

107. Ownership Conflict Resolution

Conflict resolution MUST be policy-controlled.

A generic priority rule MUST NOT be assumed.

The applicable protocol MUST define:

authority;

evidence;

ordering;

conflict resolution;

rollback;

final state.



---

108. Security Requirements

A conforming implementation MUST:

prevent unauthorized ownership;

prevent ownership forgery;

protect ownership identifiers;

prevent stale ownership reuse;

validate ownership transfer;

validate delegation;

enforce revocation;

preserve isolation;

prevent ownership-based privilege escalation;

audit ownership changes where required by the security profile.



---

109. Privacy Requirements

Ownership metadata MAY reveal:

resource relationships;

component identity;

execution structure;

system topology.


Implementations SHOULD minimize unnecessary disclosure across trust boundaries.


---

110. Auditability

High-assurance profiles SHOULD record ownership transitions:

resource
old_owner
new_owner
operation
timestamp/sequence
generation
authority
result

Audit records MUST NOT expose protected data unnecessarily.


---

111. Determinism

Ownership transitions under identical valid conditions SHOULD have deterministic semantic results.

Distributed systems MAY require explicit ordering or consensus mechanisms.


---

112. Conformance Requirements

A conforming implementation MUST demonstrate applicable ownership behavior through executable tests.

At minimum, the conformance suite SHOULD test:

1. initial ownership;


2. exclusive ownership;


3. shared ownership;


4. immutable sharing;


5. borrowing;


6. mutable borrowing;


7. transfer;


8. failed transfer;


9. stale transfer;


10. delegation;


11. delegation revocation;


12. release;


13. double release;


14. ownership generation;


15. stale ownership;


16. lifetime interaction;


17. capability interaction;


18. asynchronous ownership;


19. cancellation;


20. FFI ownership;


21. cross-process transfer;


22. distributed transfer where supported;


23. conflict detection;


24. rollback;


25. exception behavior;


26. serialization behavior;


27. cloning;


28. copying;


29. persistence;


30. device ownership where supported.




---

113. Negative Conformance

The test suite MUST deliberately attempt:

duplicate ownership;

unauthorized transfer;

unauthorized release;

unauthorized mutation;

stale ownership;

expired ownership;

revoked ownership;

transfer during conflicting borrow;

release through borrowed reference;

ownership forgery;

replayed transfer;

conflicting distributed ownership.


A conforming implementation MUST reject, detect, isolate or otherwise handle each according to the applicable contract.


---

114. Property Testing

Ownership implementations SHOULD support property-based testing of invariants.

Examples:

transfer(x, A -> B)

must never produce:

owner(x) = A AND owner(x) = B

Similarly:

release(x)

must never produce:

valid(x) = true

unless the resource contract explicitly defines release as non-invalidating.


---

115. Model Testing

Implementations SHOULD compare runtime ownership behavior against the normative state machine.

The model SHOULD verify:

legal transitions;

illegal transitions;

rollback;

transfer atomicity;

revocation;

stale authority;

concurrent transitions.



---

116. Fuzz Testing

Ownership APIs SHOULD be fuzz-tested with:

random operation sequences;

invalid owner identifiers;

stale generations;

repeated transfers;

repeated releases;

malformed capabilities;

concurrent operations;

cancellation;

failure injection.


The implementation MUST preserve ownership invariants under malformed input.


---

117. Fault Injection

The ownership implementation SHOULD support injection of:

allocation failure;

transfer failure;

process failure;

network failure;

authorization failure;

timeout;

revocation;

stale state;

conflicting state.


The resulting ownership state MUST remain valid.


---

118. Reference Implementation Requirements

A future reference implementation SHOULD provide:

OwnershipManager
OwnershipRecord
OwnershipState
OwnershipKind
OwnershipToken
OwnershipGeneration
BorrowManager
DelegationManager
TransferManager
RevocationManager
OwnershipValidator
OwnershipConflictDetector
OwnershipRecovery
OwnershipAudit

These are conceptual modules.

The reference implementation MAY use any programming language.


---

119. Required Semantic Interfaces

A ULABI ownership implementation SHOULD expose semantic interfaces equivalent to:

OwnershipAuthority
OwnershipStateProvider
OwnershipValidator
OwnershipTransfer
OwnershipBorrow
OwnershipShare
OwnershipDelegate
OwnershipRevocation
OwnershipRelease
OwnershipConflictResolver
OwnershipAudit

These interfaces are conceptual and language-independent.


---

120. Integration Contract

This document integrates with the following authoritative specifications.

120.1 docs/abi/memory-model.md

Owns:

memory-resource abstraction;

resource identity;

handles;

high-level memory ownership states;

memory boundary representation.


This document owns the detailed ownership semantics.

memory-model.md MUST NOT redefine ownership state transitions.


---

120.2 docs/memory/allocation.md

Owns:

allocation;

reallocation;

allocation failure;

deallocation mechanics;

allocator behavior.


This document defines who has authority over allocated resources.

ownership.md MUST NOT redefine allocator internals.


---

120.3 docs/memory/lifetimes.md

Owns:

validity intervals;

expiration;

lifetime extension;

borrowing duration;

destruction timing;

asynchronous retention;

lifetime revocation.


Ownership transitions MUST respect lifetime rules.

ownership.md MUST NOT redefine lifetime mechanics.


---

120.4 docs/memory/memory-safety.md

Owns:

enforcement of memory-safety invariants;

invalid ownership detection;

stale references;

unauthorized access;

safety failure behavior.


ownership.md defines the ownership contract that memory safety enforces.


---

120.5 docs/memory/shared-memory.md

Owns:

shared-memory synchronization;

memory visibility;

shared-memory ordering;

synchronization mechanisms.


Ownership determines authority; shared-memory defines concurrent memory behavior.


---

120.6 docs/memory/virtual-memory.md

Owns:

virtual address spaces;

mappings;

page semantics;

address-space behavior.


A mapping does not automatically establish ownership.


---

120.7 docs/security/capability-security.md

Owns:

capability representation;

capability delegation;

authorization;

capability revocation.


Ownership and capabilities remain separate concepts.


---

120.8 docs/security/security-model.md

Owns:

system-wide security architecture;

trust boundaries;

isolation;

security profiles.


Ownership MUST operate within those constraints.


---

120.9 docs/interoperability/foreign-function-interface.md

Owns:

FFI boundary mechanics;

language binding;

foreign calls;

cross-language ABI mapping.


Ownership semantics MUST be explicitly represented at FFI boundaries.


---

120.10 docs/abi/exception-model.md

Owns:

exception/error transport;

error representation;

propagation.


Ownership failures MUST use that common mechanism.


---

120.11 docs/abi/calling-convention.md

Owns:

parameter passing;

call frames;

calling convention.


Ownership mode MUST be represented in the interface contract without redefining the calling convention.


---

120.12 docs/abi/return-values.md

Owns:

returned-value ABI semantics.


Returned resource ownership MUST be explicitly represented.


---

120.13 Compatibility

The following own compatibility behavior:

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

This document defines which ownership semantics must remain stable.


---

120.14 Reliability

The following own failure and recovery:

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/self-healing.md

Ownership recovery MUST NOT bypass ownership invariants.


---

120.15 Standards

The following own conformance:

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md

This document defines ownership-specific requirements that those standards execute and certify.


---

121. Authoritative Definition Matrix

Concept	Authoritative file

ULABI architecture	ULABI-DESIGN.md
Normative global specification	ULABI-SPEC.md
Memory resource abstraction	docs/abi/memory-model.md
Ownership semantics	docs/memory/ownership.md
Allocation	docs/memory/allocation.md
Lifetime	docs/memory/lifetimes.md
Memory safety enforcement	docs/memory/memory-safety.md
Virtual memory	docs/memory/virtual-memory.md
Shared memory	docs/memory/shared-memory.md
Capability authorization	docs/security/capability-security.md
Security architecture	docs/security/security-model.md
FFI	docs/interoperability/foreign-function-interface.md
Calling convention	docs/abi/calling-convention.md
Returned resources	docs/abi/return-values.md
Error transport	docs/abi/exception-model.md
Compatibility	docs/compatibility/*
Reliability	docs/reliability/*
Conformance	docs/standards/*


No other document should redefine ownership semantics.


---

122. Implementation Independence

A ULABI ownership implementation MUST be possible in:

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

other languages;

Zamani;

Sankofa.


Zamani and Sankofa remain independent languages.

Neither is foundational to ULABI.

ULABI MUST NOT require either language.


---

123. Implementation Strategy

Implementations MAY map ownership using:

Static types
Runtime metadata
Reference counting
Garbage collection
Handles
Capabilities
Borrow checking
Linear types
Affine types
RAII
Reference tables
Hardware protection
Hybrid mechanisms

The implementation mechanism is not part of the universal contract.


---

124. Safe Ownership Profile

A Safe Ownership Profile SHOULD require:

explicit ownership state;

exclusive-owner enforcement where applicable;

explicit transfer;

explicit borrowing;

release authorization;

stale-authority protection;

lifetime integration;

capability integration;

deterministic failure;

conformance testing.



---

125. High-Assurance Ownership Profile

A High-Assurance Ownership Profile SHOULD additionally support:

formally defined state transitions;

invariant verification;

model checking;

property-based testing;

fault injection;

deterministic recovery;

auditability;

reproducible conformance evidence.



---

126. Ownership Safety Checklist

An implementation claiming ownership support SHOULD be able to answer:

[ ] Who owns this resource?
[ ] Can ownership be transferred?
[ ] Can ownership be shared?
[ ] Can it be borrowed?
[ ] Who may mutate it?
[ ] Who may release it?
[ ] When does ownership expire?
[ ] Can ownership be revoked?
[ ] How are stale references detected?
[ ] How are duplicate owners prevented?
[ ] How are transfer failures handled?
[ ] How does ownership cross FFI?
[ ] How does ownership cross processes?
[ ] How does ownership cross machines?
[ ] How does ownership interact with capabilities?
[ ] How does ownership interact with lifetime?
[ ] How does ownership interact with asynchronous execution?
[ ] How is ownership tested?
[ ] How is ownership certified?


---

127. Final Principle

> ULABI ownership defines who has semantic authority over a resource without requiring all languages, runtimes or systems to implement ownership in the same way.



The universal contract is:

Explicit authority
        +
Explicit transfer
        +
Explicit borrowing
        +
Explicit sharing
        +
Explicit release
        +
Lifetime consistency
        +
Capability consistency
        +
Safety enforcement
        =
Portable ownership semantics

ULABI defines the ownership contract.

Individual implementations define the mechanism.

No programming language owns ULABI.

No runtime owns ULABI.

No operating system owns ULABI.

No processor architecture owns ULABI.

No company owns the technical meaning of the ULABI ownership contract.

ULABI remains an independent interoperability standard.

## What should be done in which files

I would now freeze the dependency order like this.

### Stage 1 — foundational memory semantics

These are the files that establish independent contracts first:

| Order | File | What it owns | Depends on |
|---:|---|---|---|
| 1 | `docs/abi/memory-model.md` | Memory-resource abstraction | `ULABI-DESIGN.md` |
| 2 | `docs/memory/allocation.md` | Acquisition/reallocation/release mechanics | memory model |
| 3 | `docs/memory/ownership.md` | **Ownership authority and transitions** | memory model + allocation |
| 4 | `docs/memory/lifetimes.md` | Temporal validity | memory model + ownership |
| 5 | `docs/memory/memory-safety.md` | Enforcement of memory invariants | ownership + lifetime + allocation |
| 6 | `docs/memory/virtual-memory.md` | Address spaces/mappings | memory model |
| 7 | `docs/memory/shared-memory.md` | Sharing/synchronization | ownership + memory model |

The important correction to the previous plan is that **ownership comes before treating `memory-safety.md` as final**, because memory safety needs authoritative ownership semantics.

### Stage 2 — type and representation

```text
docs/type-system/universal-types.md
docs/type-system/type-descriptors.md
docs/type-system/type-compatibility.md
docs/type-system/structs.md
docs/type-system/unions.md
docs/type-system/enums.md
docs/type-system/generics.md
docs/abi/data-types.md

These determine what an owned resource actually contains and how its identity/type is represented.

Stage 3 — ABI boundaries

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

These determine how ownership contracts physically cross calls.

Stage 4 — interoperability

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

Stage 5 — security

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md
docs/security/cryptography.md

Stage 6 — runtime/concurrency

docs/runtime/runtime-interface.md
docs/runtime/process-model.md
docs/runtime/threading.md
docs/runtime/async-model.md
docs/runtime/concurrency.md
docs/runtime/resource-management.md

Stage 7 — reliability

docs/reliability/fault-detection.md
docs/reliability/fault-isolation.md
docs/reliability/recovery.md
docs/reliability/rollback.md
docs/reliability/health-monitoring.md
docs/reliability/self-healing.md

Stage 8 — distributed ownership/interoperability

docs/distributed/distributed-abi.md
docs/distributed/remote-calls.md
docs/distributed/serialization.md
docs/distributed/service-discovery.md
docs/distributed/distributed-errors.md
docs/distributed/consensus-boundaries.md

Stage 9 — compatibility

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Stage 10 — standards and verification

docs/standards/conformance.md
docs/standards/compliance-levels.md
docs/standards/certification.md
docs/standards/test-suite.md
docs/standards/reference-implementations.md


---

Required implementation code modules

Once those contracts are frozen, the code should be organized around semantic responsibilities, not around a particular language.

implementations/
│
├── core/
│   ├── abi/
│   ├── types/
│   ├── memory/
│   ├── ownership/
│   ├── lifetime/
│   ├── capabilities/
│   ├── errors/
│   └── versioning/
│
├── memory/
│   ├── resource
│   ├── allocator
│   ├── ownership
│   ├── lifetime
│   ├── handle
│   ├── borrow
│   ├── transfer
│   ├── sharing
│   ├── view
│   ├── bounds
│   ├── safety
│   └── domain
│
├── concurrency/
│   ├── access
│   ├── synchronization
│   ├── atomic
│   ├── ordering
│   └── race_detection
│
├── security/
│   ├── capability
│   ├── authorization
│   ├── revocation
│   └── isolation
│
├── interoperability/
│   ├── ffi
│   ├── marshal
│   ├── unmarshal
│   ├── layout
│   └── language_adapter
│
├── runtime/
│   ├── resource_manager
│   ├── ownership_manager
│   ├── lifetime_manager
│   └── async_manager
│
├── distributed/
│   ├── ownership_transfer
│   ├── resource_descriptor
│   ├── serialization
│   └── consistency
│
└── reliability/
    ├── fault_detection
    ├── fault_isolation
    ├── recovery
    ├── rollback
    └── verification

And the ownership-specific modules should be treated as:

OwnershipState
OwnershipKind
OwnershipRecord
OwnershipToken
OwnershipGeneration
OwnershipAuthority
OwnershipValidator
OwnershipManager
OwnershipTransfer
OwnershipBorrow
OwnershipShare
OwnershipDelegate
OwnershipRevocation
OwnershipRelease
OwnershipConflictDetector
OwnershipConflictResolver
OwnershipAudit
OwnershipRecovery

These should not all necessarily become separate source files. The list defines the required semantic modules; the eventual implementation language can combine or split them.

The critical dependency graph

ULABI-DESIGN.md
                        |
                   ULABI-SPEC.md
                        |
                docs/abi/memory-model.md
                        |
          +-------------+-------------+
          |             |             |
     allocation     ownership     virtual-memory
          |             |
          +------+------+ 
                 |
             lifetimes
                 |
            memory-safety
                 |
       +---------+---------+
       |                   |
   shared-memory       security
       |                   |
       +---------+---------+
                 |
              runtime
                 |
          interoperability
                 |
            distributed
                 |
          compatibility
                 |
          conformance
                 |
          certification

This is the structure that lets us work file-by-file without later discovering that another file fundamentally invalidated the previous one.

The biggest architectural rule I would now lock into the repository is:

> Every ULABI document owns one semantic domain. Other documents may reference that domain, constrain its use, or integrate with it, but they must not redefine it.



That is what will allow independent organizations—and eventually independent implementations in C, Rust, Go, Python, Zamani, Sankofa, etc.—to implement the same ULABI contract without the standard becoming coupled to any one language.