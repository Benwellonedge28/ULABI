Yes. I checked the actual Benwellonedge28/ULABI repository before drafting this. The master architecture establishes language/runtime/platform neutrality, explicit ownership and lifetime boundaries, safe zero-copy, capability security, canonical representations, profiles, and implementation independence.

I also checked the existing memory documentation. docs/memory/allocation.md is currently only a placeholder, while the repository already has separate documents for memory safety, virtual memory, shared memory, and the ABI memory model.

So allocation.md should define allocation as the ULABI resource-acquisition/release contract, without duplicating ownership, lifetime, virtual-memory, shared-memory, or general memory-safety specifications.

Below is the complete document you can copy into:

docs/memory/allocation.md


---

ULABI Memory Allocation Specification

Project: ULABI
Full Name: Universal Language Application Binary Interface
Repository: Benwellonedge28/ULABI
Document: docs/memory/allocation.md
Specification Area: Memory / Allocation
Status: Architecture / Design Specification
Specification Version: 0.2.0-draft
License: Apache-2.0


---

1. Purpose

This document defines the ULABI memory allocation contract.

The purpose of the allocation model is to provide a language-neutral and implementation-independent contract for acquiring, resizing, transferring, releasing, and validating memory resources at ULABI interoperability boundaries.

ULABI allocation must work across implementations using:

manual allocation;

garbage collection;

reference counting;

ownership systems;

arenas;

regions;

pools;

slabs;

stack allocation;

static allocation;

custom allocators;

operating-system allocators;

device memory allocators;

shared-memory allocators;

persistent-memory systems;

accelerator memory;

future memory technologies.


ULABI defines the allocation boundary contract.

It does not define one universal allocator.


---

2. Relationship to the ULABI Architecture

The master architecture establishes ULABI as a language-neutral interoperability contract rather than a universal runtime or memory-management system.

This document specializes that architecture for allocation.

The conceptual relationship is:

ULABI
                      |
              Memory Contract
                      |
              Allocation Contract
                      |
        +-------------+-------------+
        |             |             |
    Allocator      Resource      Lifetime
     Policy        Identity       Contract
        |             |             |
        +-------------+-------------+
                      |
              Implementation

Allocation is therefore separate from:

ownership;

lifetime;

virtual memory;

shared memory;

memory safety;

calling convention;

serialization.


Those specifications may depend on allocation semantics, but allocation must not redefine them.


---

3. Fundamental Principle

> ULABI standardizes the semantics of memory allocation at interoperability boundaries, not the internal allocator implementation.



A ULABI implementation may internally use any allocation strategy.

For example:

Language A
    |
    | ULABI allocation contract
    v
+-------------------+
| Allocation Layer  |
+-------------------+
    |
    +-- malloc
    +-- GC heap
    +-- arena
    +-- region
    +-- slab
    +-- pool
    +-- device allocator
    +-- custom allocator

The consumer must depend on the ULABI contract rather than the allocator's private implementation.


---

4. Non-Goals

This document does not:

1. define one universal allocator;


2. mandate malloc/free;


3. mandate garbage collection;


4. mandate reference counting;


5. mandate ownership-based languages;


6. define one heap architecture;


7. define virtual-memory management;


8. define operating-system memory management;


9. define physical memory management;


10. define general memory-safety rules;


11. define all shared-memory synchronization;


12. define serialization;


13. define distributed memory;


14. expose allocator internals;


15. require allocation for every ULABI value.




---

5. Allocation Terminology

5.1 Allocation

Creation or reservation of a memory resource according to a defined allocation contract.

5.2 Allocator

A component capable of satisfying allocation requests.

5.3 Allocation Request

A structured request describing the required memory resource.

5.4 Allocation Result

The result of an allocation operation.

It must explicitly indicate:

success;

failure;

resource identity;

size;

alignment;

permitted access;

lifetime;

ownership;

allocator/domain identity where applicable.


5.5 Memory Resource

The abstract resource produced by allocation.

5.6 Allocation Domain

A logical namespace or policy domain within which allocation and release semantics are defined.

5.7 Deallocation

The operation that ends the allocator-managed allocation according to its contract.

5.8 Reallocation

Changing the size or representation of an existing allocation according to an explicit contract.

5.9 Alignment

The required address boundary for a memory resource.

5.10 Capacity

The amount of storage available to a resource without requiring reallocation.


---

6. Allocation Contract

Every ULABI allocation interface must define:

AllocationContract {
    allocator_identity
    resource_type
    requested_size
    requested_alignment
    requested_permissions
    allocation_domain
    lifetime_policy
    ownership_policy
    initialization_policy
    failure_policy
    resizing_policy
    release_policy
}

Implementations may have additional metadata.

The fields above define the minimum semantic contract required for portable allocation behavior.


---

7. Allocation Identity

An allocation must have an unambiguous semantic identity.

A raw pointer alone is insufficient as a portable identity.

The following must not automatically be treated as ULABI allocation identities:

raw pointer
virtual address
physical address
allocator-specific address
process-local address
device-local address

An implementation may internally use any of these.

ULABI-facing resource identity must remain valid according to the applicable locality and lifetime contract.


---

8. Allocation Domains

ULABI supports explicit allocation domains.

Examples:

ProcessHeap
SharedMemoryDomain
DeviceMemoryDomain
PersistentMemoryDomain
RealTimeDomain
SecureMemoryDomain
AcceleratorDomain
CustomDomain

An allocation request may specify:

domain = ProcessHeap

or another registered allocation domain.

The domain determines which allocation and release operations are valid.


---

9. Allocator Identity

Where multiple allocators coexist, an allocator identity should be available.

Conceptually:

AllocatorIdentity {
    namespace
    identifier
    version
    capabilities
}

Allocator identity must not expose implementation details unnecessarily.

It exists primarily to ensure that resources are released through compatible mechanisms.


---

10. Allocation Ownership

Allocation and ownership are related but distinct concepts.

Allocation answers:

> Who created or reserved the memory resource?



Ownership answers:

> Who is responsible for controlling and releasing the resource?



A component may allocate memory and subsequently transfer ownership.

Example:

Allocator A
    |
    | allocate
    v
Resource R
    |
    | ownership transfer
    v
Consumer B

The ownership model itself is defined by the ULABI memory model.

This document only defines the allocation operations that create and release resources.


---

11. Initialization

Every allocation interface must specify whether allocated memory is initialized.

Supported semantic modes may include:

Uninitialized
ZeroInitialized
PatternInitialized
TypeInitialized
ImplementationInitialized

An implementation must never silently treat uninitialized memory as initialized semantic data.

If initialization is unspecified, the interface must define a safe default.


---

12. Zero Initialization

Where requested, zero initialization must produce a deterministic zeroed representation appropriate to the allocation contract.

For raw byte storage:

byte[i] = 0

For typed objects, zero initialization must not automatically imply a valid semantic value unless the type contract explicitly permits it.

For example:

zero bytes != automatically valid object


---

13. Allocation Size

Allocation size must be represented independently of the implementation's native integer size.

The contract must define:

requested_size
allocated_size
usable_size
capacity

where applicable.

The implementation must reject requests that exceed supported limits.

It must not silently wrap sizes.


---

14. Integer Overflow

Allocation-size arithmetic must be overflow-safe.

For example:

element_count * element_size

must be checked before allocation.

If the calculation cannot be represented safely, allocation must fail with a defined error.

It must never wrap into a smaller allocation.


---

15. Alignment

Allocation requests may specify alignment.

Example:

allocate(
    size = 4096,
    alignment = 64
)

The returned resource must satisfy the declared alignment requirement.

If the allocator cannot provide the requested alignment, it must return a defined allocation failure.

It must not silently provide weaker alignment.


---

16. Alignment Requirements

ULABI alignment values must be:

explicitly represented;

validated;

compatible with the target allocation domain;

within implementation limits.


An implementation may support stronger alignment than requested.

It must not provide weaker alignment while reporting success.


---

17. Memory Permissions

An allocation may declare permissions:

Read
Write
Execute
Atomic
DeviceAccess
DMA
Shared

Permissions must be explicit where they affect safety or security.

Executable memory should require an explicit policy.

Writable-and-executable memory must not be the default.


---

18. W^X Compatibility

Where supported by the platform, ULABI implementations should follow a write-xor-execute policy:

Writable + Executable

should require explicit authorization.

Allocation APIs must not silently create executable writable memory merely for convenience.


---

19. Secure Allocation

Security-sensitive allocations may request:

SecureMemory
NonDumpable
NonSwappable
ZeroOnRelease
Protected
Isolated

These are profile-dependent capabilities.

An implementation that cannot satisfy a mandatory security property must reject the request rather than silently downgrade it.


---

20. Allocation Failure

Allocation failure must be explicit.

Possible failure categories include:

OutOfMemory
SizeOverflow
InvalidAlignment
UnsupportedAlignment
InvalidDomain
PermissionDenied
ResourceLimit
AddressSpaceExhausted
Fragmentation
DeviceUnavailable
QuotaExceeded
SecurityPolicyViolation
UnsupportedFeature
AllocatorUnavailable

The exact error taxonomy may be extended through ULABI error mechanisms.


---

21. No Silent Fallback

An allocator must not silently change the requested semantics.

For example:

requested: SecureMemory
provided: ordinary memory

must not be reported as success.

Likewise:

requested: 64-byte alignment
provided: 16-byte alignment

must not be reported as successful allocation.


---

22. Resource Limits

ULABI implementations may impose explicit resource limits.

Examples:

maximum allocation size
maximum allocations
maximum total memory
maximum alignment
maximum allocation rate
maximum domain quota

Limits should be discoverable or reported through defined capability/error mechanisms.


---

23. Allocation Quotas

An allocation domain may have a quota.

Example:

DomainQuota {
    max_bytes
    current_bytes
    max_allocations
    current_allocations
}

A quota violation must produce a defined failure.

Quota enforcement must not corrupt existing allocations.


---

24. Determinism

ULABI allocation semantics should be deterministic at the contract level.

The physical address returned by an allocator does not need to be deterministic.

However, given equivalent conditions, the observable contract must remain predictable.

For example:

same request
+
same policy
+
sufficient resources

must not produce semantically incompatible results.


---

25. Address Independence

Portable ULABI code must not depend on a particular allocation address.

For example:

0x10000000

has no universal ULABI meaning.

Addresses are implementation-local unless an explicit address-space contract states otherwise.


---

26. Allocation Handle

Where raw pointers are inappropriate, allocation may return an opaque handle.

Conceptually:

AllocationHandle<T>

The handle may contain or reference:

resource_id
generation
domain
permissions
lifetime

The internal representation is implementation-defined.


---

27. Generation Protection

Implementations may use generation counters to detect stale allocation handles.

Example:

resource_id = 42
generation = 7

After release and reuse:

resource_id = 42
generation = 8

A stale handle containing generation 7 must not access the new resource.


---

28. Deallocation

Deallocation ends the allocation according to its contract.

Conceptually:

deallocate(resource)

The operation must validate:

resource identity;

ownership;

allocation domain;

allocator compatibility;

resource state;

permissions.


Unauthorized deallocation must fail safely.


---

29. Matching Allocation and Deallocation

Unless explicitly specified otherwise:

> A resource must be released through a compatible allocation domain and release contract.



This prevents incompatible allocator pairs such as:

Allocator A
    |
    | allocate
    v
Memory
    |
    | incompatible release
    v
Allocator B

The ULABI contract must make such compatibility explicit.


---

30. Double Deallocation

A resource must not be successfully deallocated twice under the same ownership contract.

The second operation must either:

be prevented;

return an already-released error;

be handled as an idempotent release if the contract explicitly defines that behavior.


It must never corrupt unrelated allocations.


---

31. Use After Deallocation

After successful deallocation, the resource becomes invalid according to its lifetime contract.

Subsequent access must fail safely where the implementation can detect it.

The allocation contract must not permit a deallocated resource to become silently valid again under the same identity.


---

32. Reallocation

ULABI may support:

reallocate(resource, new_size)

Reallocation must explicitly define whether:

the original resource remains valid;

the resource may move;

existing contents are preserved;

ownership changes;

alignment is preserved;

permissions are preserved;

failure leaves the original allocation unchanged.



---

33. Failure-Atomic Reallocation

Unless explicitly documented otherwise:

> A failed reallocation must leave the original valid resource unchanged.



Conceptually:

Old Resource
     |
     | reallocate
     v
Attempt
  /   \
success failure
 |       |
New     Old remains valid

This prevents loss of the original resource on allocation failure.


---

34. Growth

Growing an allocation may require relocation.

Therefore callers must not assume that:

old_address == new_address

unless the contract explicitly guarantees in-place growth.


---

35. Shrinking

Shrinking an allocation must define:

whether contents outside the new size become inaccessible;

whether capacity changes;

whether the allocation moves;

whether alignment remains valid.


Access beyond the new logical size must not be considered valid.


---

36. Preservation During Reallocation

Where reallocation guarantees preservation, bytes in the preserved range must remain semantically unchanged.

For:

old_size = N
new_size = M

the preserved range is:

0 .. min(N, M)

unless the type contract defines a different semantic transformation.


---

37. Allocation Transfer

Allocation resources may be transferred between components.

Example:

A
|
| allocation
v
R
|
| transfer
v
B

Transfer must integrate with the ownership and lifetime contracts.

After successful transfer, the previous owner must not continue using the transferred ownership.


---

38. Borrowed Allocations

A component may expose allocated memory as a borrow rather than transferring ownership.

Example:

Allocator
   |
   v
Owner
   |
   | borrow
   v
Consumer

The consumer must not deallocate borrowed memory unless the contract explicitly grants that capability.


---

39. Allocation and Copying

Allocation may be used to create a destination for a copy.

Conceptually:

Source
  |
  | copy
  v
Destination Allocation

The copy operation must define:

source validity;

destination size;

overlap behavior;

type semantics;

failure behavior;

ownership of the destination.



---

40. Allocation and Zero-Copy

ULABI supports zero-copy where safe.

Zero-copy means that a consumer accesses an existing resource without creating an independent copy.

This requires explicit:

ownership;

borrowing;

lifetime;

access;

synchronization;

locality;

capability semantics.


Zero-copy must never mean:

> "Ignore memory ownership because copying is expensive."




---

41. Allocation and Memory Views

A view may reference an existing allocation.

Example:

Allocation
    |
    +-------------------+
    |                   |
    v                   v
View A                View B

The underlying allocation must remain valid for every active view.

View semantics are defined by the memory model.

Allocation is responsible only for the underlying resource.


---

42. Allocation and Shared Memory

Shared-memory allocation must identify its allocation domain.

For example:

SharedMemoryDomain

The allocation contract must not assume that ordinary process-local allocation is shareable.

Shared-memory semantics are further defined by:

docs/memory/shared-memory.md


---

43. Allocation and Virtual Memory

Allocation is distinct from virtual-memory mapping.

Conceptually:

Allocation
    |
    v
Memory Resource
    |
    v
Virtual Mapping
    |
    v
Address Space

The virtual-memory specification determines mapping semantics.

This document determines allocation semantics.


---

44. Allocation and Physical Memory

ULABI does not require allocation APIs to expose physical addresses.

Physical-memory access requires an explicit platform/hardware profile and appropriate capabilities.

A virtual allocation must not be assumed to correspond to contiguous physical memory.


---

45. Device Allocation

Device-specific allocation may expose resources such as:

GPUBuffer
NPUBuffer
FPGARegion
DMARegion
QuantumResource

Such resources must declare their domain and capabilities.

The general allocation contract remains language-neutral.

Device-specific behavior belongs to hardware/platform profiles.


---

46. DMA and Device Access

Memory intended for direct device access may require properties such as:

DMACompatible
Pinned
Coherent
NonCoherent
DeviceVisible

These properties must be explicit.

An implementation must not claim device compatibility unless the requested property is actually satisfied.


---

47. Persistent Memory

Persistent-memory allocations may have additional semantics:

Persistent
Durable
Flushable
Recoverable

Persistence must not be inferred from ordinary allocation.

The allocation contract must distinguish:

volatile allocation

from:

persistent allocation


---

48. Real-Time Allocation

Real-time profiles may prohibit unbounded allocation latency.

A real-time allocation contract may specify:

bounded_latency
preallocated_pool
no_blocking
no_page_fault
fixed_capacity

Failure must remain explicit.

ULABI must not promise real-time guarantees merely because an allocator exists.


---

49. Allocation and Concurrency

Allocators may be:

SingleThreaded
ThreadSafe
LockFree
WaitFree
ExternallySynchronized

The concurrency property must be declared.

Thread safety of the allocator does not automatically make the allocated object thread-safe.


---

50. Allocation and Async Operations

Asynchronous allocation may be supported through an extension profile.

Example:

allocate_async(request) -> Future<Allocation>

The future must define:

cancellation;

failure;

ownership;

lifetime;

completion semantics.


The resource must not become accessible before successful completion.


---

51. Allocation and Cancellation

Cancellation must define whether an in-progress allocation:

is guaranteed to stop;

may complete later;

can be safely abandoned;

produces a resource;

releases any partial resources.


Cancellation must not create leaked resources.


---

52. Allocation and Exceptions

Allocation failures must integrate with ULABI's error model.

An implementation may map:

OutOfMemory

to its native error mechanism.

However, the ULABI boundary must preserve the semantic error.

A language-specific exception must not be required for interoperability.


---

53. Allocation and Capabilities

Allocation must integrate with capability security.

An allocation capability may restrict:

maximum_size
domains
permissions
alignment
device_access
sharing
persistence
executable_memory

Possession of an allocation capability must not automatically grant unrelated system capabilities.


---

54. Sandboxing

A sandboxed implementation may expose only a restricted allocator.

For example:

SandboxAllocator {
    max_bytes = ...
    allowed_domains = ...
    allowed_permissions = ...
}

Attempts outside the sandbox must fail according to the security contract.


---

55. Resource Accounting

Implementations should be able to account for:

allocated_bytes
released_bytes
active_allocations
peak_usage
allocation_failures
allocation_latency

Observability semantics belong to the observability specifications.

Allocation implementations should expose these metrics through the appropriate interface rather than redefining them here.


---

56. Leak Detection

Implementations may detect allocation leaks.

A leak occurs when:

allocation remains active
+
no valid ownership/reference path remains

Leak detection is primarily an implementation and diagnostic concern.

ULABI does not require one leak-detection algorithm.


---

57. Leak Prevention

A conforming implementation must not intentionally leak resources as a normal successful allocation strategy.

Where release is impossible due to process termination, the implementation may rely on the operating system or runtime to reclaim process-local resources.

Externally persistent resources require explicit recovery semantics.


---

58. Allocator Failure Recovery

If an allocator fails:

Allocation requested
       |
       v
    Failure
       |
       +--> report error
       |
       +--> preserve existing resources
       |
       +--> no partial ownership

A failed allocation must not create an ambiguously owned resource.


---

59. Partial Allocation

If an operation requires multiple allocations:

A
B
C
D

and allocation C fails, the implementation must have a defined rollback strategy.

Unless explicitly specified otherwise:

A -> release
B -> release
C -> failed
D -> not created

The operation must not leave hidden partially owned resources.


---

60. Transactional Allocation

Complex allocation operations may use transactional semantics.

Conceptually:

Begin
  |
  +-- allocate A
  +-- allocate B
  +-- allocate C
  |
Commit

On failure:

Rollback

Transactional allocation is optional but useful for composite resources.


---

61. Fragmentation

Allocation failure may occur even when aggregate free memory is sufficient because the requested contiguous region cannot be provided.

The implementation should distinguish, where observable and safe:

OutOfMemory
AddressSpaceExhausted
Fragmentation

However, implementations are not required to expose internal allocator diagnostics that would create security or portability problems.


---

62. Contiguous Allocation

A request for contiguous memory must explicitly state the relevant address space.

For example:

ContiguousVirtualMemory

does not necessarily imply:

ContiguousPhysicalMemory

These are separate properties.


---

63. Pool Allocation

A pool allocator may provide:

allocate_from_pool(pool, size)

The pool identity becomes part of the allocation domain.

Resources must be released according to the pool's contract.


---

64. Arena Allocation

Arena allocation may use:

Arena
    |
    +-- allocation A
    +-- allocation B
    +-- allocation C

An arena may release many allocations together.

The contract must explicitly define the lifetime relationship between the arena and its allocations.

An individual allocation must not be assumed to outlive its arena.


---

65. Region Allocation

Region-based allocation follows similar semantics.

A region may provide a bulk-release operation.

If an individual object requires independent lifetime, it must be moved, copied, retained, or transferred into an appropriate independent allocation domain.


---

66. Stack Allocation

Stack-backed memory may be exposed only when the lifetime contract guarantees validity.

A stack allocation must not escape its valid scope unless explicitly copied or transferred into a longer-lived resource.

Example:

Call
 |
 +-- stack allocation
 |
 +-- return
      |
      X invalid

Returning a raw reference to expired stack memory is non-conforming.


---

67. Static Allocation

Static memory may have process or module lifetime.

Its ownership and mutability must still be explicit when exposed through ULABI.

Static allocation does not automatically imply thread safety.


---

68. Garbage-Collected Memory

A garbage-collected implementation may expose memory through:

copies;

pinned regions;

stable handles;

managed references;

explicit foreign-memory regions.


ULABI must not require another implementation to understand the garbage collector.

A foreign component must not directly manipulate a GC-managed object unless the contract explicitly supports it.


---

69. Moving Garbage Collectors

A moving garbage collector may relocate memory.

Therefore raw pointers into managed memory must not be assumed stable unless the allocation contract guarantees stability.

Stable handles or pinned allocations should be used where required.


---

70. Reference Counting

Reference-counted implementations may use reference counts internally.

The reference count itself is not part of the portable ULABI allocation contract unless explicitly exposed.


---

71. Custom Allocators

ULABI implementations may provide custom allocators.

A custom allocator must expose enough semantic information to determine:

supported sizes;

alignment;

domains;

permissions;

ownership;

release mechanism;

supported operations;

failure behavior.



---

72. Allocator Compatibility

Two allocators are compatible only when their contracts explicitly permit resources to move between them.

Compatibility must not be inferred merely because:

size == size

or:

pointer type == pointer type


---

73. Allocator Replacement

An implementation may replace its internal allocator without changing its ULABI interface if the externally visible allocation contract remains compatible.

This is an important implementation-independence requirement.

For example:

ULABI
 |
 +-- Allocator v1
 |
 +-- Allocator v2
 |
 +-- Allocator v3

must be possible without changing consumers.


---

74. Allocation ABI Stability

The ULABI allocation interface must remain stable across implementation versions.

Changes to:

allocator identifiers;

resource semantics;

alignment guarantees;

release semantics;

ownership;

initialization;

error behavior


must follow ULABI versioning and compatibility rules.


---

75. Versioned Allocation Contracts

Allocation interfaces should carry or reference:

interface_id
contract_version
allocator_capabilities

Consumers must be able to determine whether the required allocation semantics are supported.


---

76. Capability Discovery

An allocator should expose discoverable capabilities such as:

supports_alignment
supports_zero_init
supports_resize
supports_secure_memory
supports_shared_memory
supports_device_memory
supports_persistence
supports_async
supports_realtime

Unsupported mandatory capabilities must result in explicit failure.


---

77. Feature Negotiation

Allocation features may be negotiated where multiple implementations interact.

Example:

Consumer:
    requires 64-byte alignment
    requires zero initialization

Allocator:
    supports 64-byte alignment
    supports zero initialization

Result:
    compatible

Negotiation semantics belong to the compatibility layer.

Allocation defines the capabilities being negotiated.


---

78. Cross-Process Allocation

A process-local allocation must not automatically become valid in another process.

Cross-process allocation requires an explicit mechanism such as:

shared memory;

exported resource handles;

serialized copies;

operating-system IPC resources.



---

79. Distributed Allocation

A remote memory resource must not be represented as though it were ordinary local memory unless an explicit distributed-memory profile defines the semantics.

Network failure, latency, revocation, and consistency must remain visible through the appropriate distributed contract.


---

80. Security Invariants

A conforming allocation implementation must enforce:

1. no unauthorized allocation capability;


2. no unauthorized deallocation;


3. no silent permission escalation;


4. no silent alignment downgrade;


5. no silent security-property downgrade;


6. no integer-overflow allocation;


7. no double-release corruption;


8. no cross-domain release unless authorized;


9. no stale-handle reuse;


10. no execution permission by implicit default.




---

81. Safety Invariants

The following invariants are normative:

I1: Successful allocation returns a valid resource.

I2: Failed allocation does not create ambiguous ownership.

I3: Requested mandatory alignment is satisfied.

I4: Requested mandatory permissions are satisfied.

I5: Allocation size cannot wrap.

I6: Deallocation invalidates the resource according to contract.

I7: A resource cannot be successfully released twice.

I8: A resource cannot be released by an unauthorized component.

I9: Reallocation failure preserves the original resource unless explicitly specified otherwise.

I10: Allocation domains remain explicit.

I11: Raw addresses are not universal resource identities.

I12: Initialization state is explicit.

I13: Security-sensitive allocation properties cannot be silently downgraded.

I14: Lifetime remains explicit.

I15: Allocator replacement must not break a compatible ULABI contract.


---

82. Failure Model

Allocation failures must be represented using ULABI's common error model.

Minimum semantic categories:

AllocationError
    |
    +-- OutOfMemory
    +-- SizeOverflow
    +-- InvalidAlignment
    +-- UnsupportedAlignment
    +-- InvalidDomain
    +-- PermissionDenied
    +-- QuotaExceeded
    +-- AddressSpaceExhausted
    +-- DeviceUnavailable
    +-- UnsupportedCapability
    +-- AllocatorUnavailable
    +-- SecurityViolation
    +-- ResourceLimit

Implementations may extend the error set.


---

83. Recovery

Allocation recovery must be policy-controlled.

Possible recovery strategies include:

Retry
UseAlternativeAllocator
ReduceRequest
ReleaseTemporaryResources
FallbackToCopy
FallbackToDifferentDomain
Escalate

A fallback may only be used if it preserves the required semantic contract.


---

84. No Unsafe Fallback

For example:

requested secure memory
        |
        X
ordinary memory fallback

is prohibited when secure memory was mandatory.

Likewise:

requested shared memory
        |
        X
private memory fallback

must not be reported as equivalent.


---

85. Allocation Lifecycle

The canonical lifecycle is:

Request
   |
Validate
   |
Authorize
   |
Allocate
   |
Initialize
   |
Publish
   |
Use / Borrow / Transfer
   |
Resize or Copy if required
   |
Release
   |
Invalidate

Failure at any stage must produce defined behavior.


---

86. Reference State Machine

+-------------+
                 |   Requested |
                 +------+------+
                        |
                     validate
                        |
                        v
                 +-------------+
                 |  Allocating |
                 +------+------+
                        |
             +----------+----------+
             |                     |
          success                failure
             |                     |
             v                     v
       +-----------+         +-----------+
       |  Active   |         |   Failed  |
       +-----+-----+         +-----------+
             |
       +-----+----------+
       |                |
    transfer          release
       |                |
       v                v
  +---------+       +-----------+
  | Active  |       | Released  |
  | NewOwner|       +-----------+
  +---------+


---

87. Reference Interface

A language-neutral conceptual interface is:

interface Allocator {

    capabilities() -> AllocatorCapabilities

    allocate(
        request: AllocationRequest
    ) -> Result<Allocation, AllocationError>

    deallocate(
        allocation: Allocation
    ) -> Result<Unit, AllocationError>

    reallocate(
        allocation: Allocation,
        request: ResizeRequest
    ) -> Result<Allocation, AllocationError>

    validate(
        allocation: Allocation
    ) -> Result<AllocationState, AllocationError>
}

This is a semantic interface.

It is not a required source-language API.


---

88. Allocation Request

Conceptually:

AllocationRequest {
    size
    alignment
    domain
    permissions
    initialization
    lifetime
    ownership
    capabilities
    flags
}

Implementations may extend this structure.

Unknown mandatory fields must not be ignored.


---

89. Allocation Result

Conceptually:

Allocation {
    resource_id
    size
    alignment
    domain
    permissions
    ownership
    lifetime
    state
}

The physical address, allocator internals, and runtime-specific metadata remain implementation-defined.


---

90. Resize Request

Conceptually:

ResizeRequest {
    new_size
    alignment_policy
    preservation_policy
    relocation_policy
}


---

91. Conformance Requirements

An implementation claiming allocation-profile conformance must demonstrate:

Basic

allocation;

deallocation;

size validation;

alignment;

initialization;

failure reporting.


Safety

overflow protection;

double-release protection;

stale-resource protection;

ownership enforcement;

lifetime enforcement.


Compatibility

stable allocation identifiers;

allocator-domain compatibility;

versioned contracts;

capability discovery.


Security

permission enforcement;

capability enforcement;

no unauthorized deallocation;

no silent security downgrade.


Advanced

Where claimed:

reallocation;

shared allocation;

secure allocation;

device allocation;

persistent allocation;

asynchronous allocation;

real-time allocation.



---

92. Required Conformance Tests

The test suite must eventually include:

allocation_basic
allocation_zero_size
allocation_large_size
allocation_overflow
allocation_alignment
allocation_invalid_alignment
allocation_zero_initialized
allocation_uninitialized
allocation_permission
allocation_domain
allocation_quota
allocation_out_of_memory
allocation_deallocation
allocation_double_deallocation
allocation_stale_handle
allocation_reallocation_success
allocation_reallocation_failure
allocation_reallocation_preservation
allocation_transfer
allocation_borrow
allocation_secure
allocation_shared
allocation_device
allocation_async
allocation_capability_discovery
allocation_versioning
allocation_allocator_replacement

Tests must verify semantic behavior rather than a particular allocator implementation.


---

93. Fuzz Testing

Allocation interfaces should be fuzz-tested with:

random sizes;

boundary sizes;

alignment values;

malformed handles;

invalid domains;

repeated release;

repeated resize;

concurrent operations;

cancellation;

capability combinations.


Fuzzing must verify that malformed requests cannot corrupt allocator state.


---

94. Stress Testing

Stress tests should evaluate:

high allocation rate
high deallocation rate
fragmentation
large allocations
small allocations
mixed sizes
mixed alignment
concurrent allocation
quota pressure
failure recovery

The tests must remain implementation-independent wherever possible.


---

95. Security Testing

Security tests must attempt:

unauthorized allocation
unauthorized release
cross-domain release
stale handle use
permission escalation
executable-memory escalation
size overflow
alignment bypass
quota bypass
capability forgery

All must fail safely.


---

96. Integration With Other ULABI Documents

This document deliberately establishes its dependencies now so later documents do not require retroactive redesign.

docs/abi/memory-model.md

Defines:

ownership;

borrowing;

lifetime;

memory resources;

validity;

transfer;

memory views.


allocation.md creates and releases the resources defined there.

docs/memory/memory-safety.md

Defines general safety guarantees.

allocation.md supplies allocation-specific safety invariants.

docs/memory/ownership.md

Defines ownership semantics.

Allocation uses those ownership states but does not redefine them.

docs/memory/lifetimes.md

Defines lifetime semantics.

Allocation requests reference those lifetime contracts.

docs/memory/virtual-memory.md

Defines address-space and mapping semantics.

Allocation does not define virtual mappings.

docs/memory/shared-memory.md

Defines shared-memory semantics.

Allocation may create resources in shared-memory domains.

docs/abi/data-types.md

Defines portable ULABI data types.

Allocation may provide storage for those types.

docs/abi/calling-convention.md

Defines function-call boundary behavior.

Allocation interfaces may be invoked through the calling convention.

docs/abi/exception-model.md

Defines error/exception boundary behavior.

Allocation failures must use that model.

docs/interoperability/cross-language-data.md

Defines cross-language data exchange.

Allocation may provide backing storage for cross-language data.

docs/interoperability/foreign-function-interface.md

Defines FFI interaction.

Allocation functions may be exposed through FFI.

docs/runtime/resource-management.md

Defines runtime resource management.

Allocation is one category of managed resource.

docs/security/capability-security.md

Defines capability-based authority.

Allocation capabilities must obey that model.

docs/security/sandboxing.md

Defines isolation.

Allocation domains may be sandbox-restricted.

docs/compatibility/feature-negotiation.md

Defines negotiation.

Allocator capabilities may be negotiated through that mechanism.

docs/distributed/serialization.md

Defines serialized representation.

Allocation may provide temporary or persistent buffers for serialization.

docs/hardware/*

Hardware-specific allocation semantics extend this document rather than replacing it.


---

97. Required Schemas

The following schemas should eventually exist:

schemas/allocation-request.schema.json
schemas/allocation-result.schema.json
schemas/allocation-error.schema.json
schemas/allocation-capabilities.schema.json
schemas/allocation-domain.schema.json
schemas/resize-request.schema.json
schemas/allocation-contract.schema.json

These schemas must encode the normative fields without tying ULABI to a programming language.


---

98. Required Examples

The repository should eventually contain:

examples/allocation/
├── basic-allocation/
├── aligned-allocation/
├── zero-initialized/
├── ownership-transfer/
├── borrowed-allocation/
├── reallocation/
├── secure-allocation/
├── shared-allocation/
├── device-allocation/
├── arena-allocation/
├── pool-allocation/
└── failure-recovery/

Examples should demonstrate contracts, not prescribe one language.


---

99. Required Implementation Modules

The allocation subsystem should eventually be organized around these language-neutral responsibilities.

allocation/
├── mod
├── allocator
├── request
├── result
├── resource
├── handle
├── domain
├── capabilities
├── permissions
├── alignment
├── initialization
├── limits
├── quota
├── deallocation
├── reallocation
├── transfer
├── validation
├── errors
├── lifecycle
└── policy

Recommended concrete module/file names:

src/allocation/mod.*
src/allocation/allocator.*
src/allocation/request.*
src/allocation/result.*
src/allocation/resource.*
src/allocation/handle.*
src/allocation/domain.*
src/allocation/capabilities.*
src/allocation/permissions.*
src/allocation/alignment.*
src/allocation/initialization.*
src/allocation/limits.*
src/allocation/quota.*
src/allocation/deallocation.*
src/allocation/reallocation.*
src/allocation/transfer.*
src/allocation/validation.*
src/allocation/errors.*
src/allocation/lifecycle.*
src/allocation/policy.*

The extension point for implementations should additionally support:

src/allocation/backends/
├── process.*
├── shared_memory.*
├── device.*
├── persistent.*
├── secure.*
├── realtime.*
└── custom.*

These are implementation architecture names, not requirements that every language use the same source-language structure.


---

100. Reference Implementation Modules

A reference implementation should provide:

reference/allocation/
├── allocator
├── request
├── resource
├── handle
├── domain
├── capabilities
├── deallocation
├── reallocation
├── validation
├── errors
└── tests

The reference implementation must be deliberately simple and auditable.

It must not become the definition of ULABI.

The specification remains authoritative.


---

101. Required Test Modules

The conformance implementation should eventually contain:

tests/allocation/
├── basic
├── alignment
├── initialization
├── limits
├── quota
├── failure
├── deallocation
├── reallocation
├── ownership
├── lifetime
├── handles
├── security
├── concurrency
├── shared
├── device
└── compatibility


---

102. Required Integration Order

To satisfy the requirement that each file can be completed without repeatedly rewriting it later, the allocation subsystem should be implemented in this order:

Phase 1 — Independent contracts

1. schemas/allocation-request.schema.json


2. schemas/allocation-capabilities.schema.json


3. schemas/allocation-domain.schema.json


4. schemas/allocation-error.schema.json


5. docs/memory/allocation.md



These establish the allocation vocabulary and contracts.

Phase 2 — Core implementation primitives

6. allocation/request


7. allocation/capabilities


8. allocation/domain


9. allocation/permissions


10. allocation/alignment


11. allocation/initialization


12. allocation/errors



Phase 3 — Resource lifecycle

13. allocation/resource


14. allocation/handle


15. allocation/allocator


16. allocation/result


17. allocation/validation


18. allocation/lifecycle



Phase 4 — Mutation and release

19. allocation/deallocation


20. allocation/reallocation


21. allocation/transfer



Phase 5 — Resource policy

22. allocation/limits


23. allocation/quota


24. allocation/policy



Phase 6 — Backends

25. allocation/backends/process


26. allocation/backends/shared_memory


27. allocation/backends/device


28. allocation/backends/persistent


29. allocation/backends/secure


30. allocation/backends/realtime


31. allocation/backends/custom



Phase 7 — Integration

32. ABI memory model


33. ownership


34. lifetimes


35. memory safety


36. shared memory


37. virtual memory


38. FFI


39. security capabilities


40. runtime resource management


41. compatibility negotiation



Phase 8 — Verification

42. allocation unit tests


43. conformance tests


44. fuzz tests


45. stress tests


46. security tests


47. cross-language tests




---

103. Integration Rule

The dependency direction must remain:

ULABI Core
    |
    v
Memory Contract
    |
    v
Allocation Contract
    |
    +----------------+
    |                |
    v                v
Allocator        Resource
    |                |
    +-------+--------+
            |
            v
       Implementation

Higher-level systems must depend on allocation semantics.

Allocation must not depend on a particular language, runtime, operating system, CPU, compiler, or allocator implementation.


---

104. What This File Owns

docs/memory/allocation.md owns:

allocation requests;

allocation results;

allocator identity;

allocation domains;

allocation capabilities;

allocation limits;

quotas;

initialization;

alignment;

permissions;

allocation failure;

deallocation;

reallocation;

allocation lifecycle;

allocator compatibility;

allocator replacement;

allocation-specific conformance.



---

105. What This File Does Not Own

It deliberately does not own:

general ownership semantics;

general lifetime semantics;

virtual-memory mapping;

shared-memory synchronization;

general memory safety;

CPU register allocation;

stack-frame layout;

serialization;

network transport;

language-specific garbage collection;

language-specific pointer semantics.


Those belong to their dedicated specifications.


---

106. Final Architectural Rule

ULABI allocation must follow this principle:

> ULABI defines how memory resources are acquired and released across an interoperability boundary; it does not dictate how any implementation manages memory internally.



Therefore:

C allocator
C++ allocator
Rust allocator
Go runtime
Java GC
Python runtime
Swift ARC
Zamani runtime
Sankofa runtime
custom embedded allocator
GPU allocator
NPU allocator
future memory system

may all independently implement the ULABI allocation contract.

ULABI remains independent of all of them.


---

107. Completion Criteria

docs/memory/allocation.md is considered complete for this specification version when:

allocation semantics are formally defined;

failure behavior is defined;

initialization is defined;

alignment is defined;

permission semantics are defined;

allocator/domain identity is defined;

deallocation semantics are defined;

reallocation semantics are defined;

ownership/lifetime integration is specified;

security integration is specified;

compatibility integration is specified;

conformance requirements are specified;

required schemas are identified;

required tests are identified;

required implementation modules are identified;

dependency boundaries are explicit.


No later ULABI document should need to redefine allocation semantics.


---

What should be done under which files

For this particular subsystem, I would lock the ownership of the work as follows:

File/module	Responsibility	Depends on

docs/memory/allocation.md	Normative allocation contract	ULABI architecture
schemas/allocation-request.schema.json	Request structure	allocation spec
schemas/allocation-result.schema.json	Result structure	allocation spec
schemas/allocation-error.schema.json	Allocation errors	allocation spec
schemas/allocation-capabilities.schema.json	Capability declaration	allocation spec
schemas/allocation-domain.schema.json	Allocation domains	allocation spec
schemas/resize-request.schema.json	Resize contract	allocation spec
schemas/allocation-contract.schema.json	Formal contract metadata	allocation spec
src/allocation/request.*	Validate/represent requests	request schema
src/allocation/capabilities.*	Capability model	capability schema
src/allocation/domain.*	Allocation domains	domain schema
src/allocation/permissions.*	Access permissions	security contract
src/allocation/alignment.*	Alignment validation	request
src/allocation/initialization.*	Initialization policy	request
src/allocation/errors.*	Allocation errors	error schema
src/allocation/resource.*	Resource state/identity	memory model
src/allocation/handle.*	Opaque resource handles	resource
src/allocation/allocator.*	Allocator interface	request/resource
src/allocation/result.*	Allocation results	resource/errors
src/allocation/validation.*	Contract validation	all core allocation primitives
src/allocation/lifecycle.*	State transitions	resource/validation
src/allocation/deallocation.*	Release	lifecycle/domain
src/allocation/reallocation.*	Resize	allocator/resource
src/allocation/transfer.*	Transfer	ownership/lifetime
src/allocation/limits.*	Resource limits	request/domain
src/allocation/quota.*	Accounting/quota	domain/limits
src/allocation/policy.*	Policy-controlled allocation/recovery	security/runtime
src/allocation/backends/process.*	Process-local backend	allocator
src/allocation/backends/shared_memory.*	Shared-memory backend	shared-memory spec
src/allocation/backends/device.*	Device backend	hardware profiles
src/allocation/backends/persistent.*	Persistent backend	persistence profile
src/allocation/backends/secure.*	Secure-memory backend	security profile
src/allocation/backends/realtime.*	Bounded-latency backend	real-time profile
src/allocation/backends/custom.*	External/custom allocator adapter	allocator contract
tests/allocation/*	Behavioral tests	implementation
conformance/allocation/*	Formal conformance	specification + tests
reference/allocation/*	Minimal reference implementation	specification


Most important architectural decision

Do not make allocation.md a giant description of malloc, garbage collection, arenas, or one language's allocator.

The repository already has a dedicated ABI memory model and dedicated memory documents. The clean separation is:

ULABI Memory Architecture
                         |
        +----------------+----------------+
        |                |                |
     Ownership        Lifetime        Allocation
        |                |                |
        +----------------+----------------+
                         |
                  Memory Resource
                         |
          +--------------+--------------+
          |              |              |
       Process        Shared         Device
       Memory         Memory         Memory

That keeps allocation.md independently completable while giving every future file a precise integration boundary. It also preserves the central ULABI rule: Zamani and Sankofa may implement this contract independently, but neither becomes part of the ULABI specification itself.