I inspected the actual Benwellonedge28/ULABI repository before drafting this. The current ULABI-DESIGN.md already establishes ULABI as language-, runtime-, platform-, architecture-, and vendor-neutral, with explicit ownership/lifetime boundaries, safe memory interoperability, zero-copy where safe, capability security, deterministic behavior, profiles, and conformance.

I also checked the existing memory-related specifications. docs/abi/memory-model.md already defines the general memory-resource, ownership, lifetime, address-space, layout, and zero-copy boundary model; docs/memory/memory-safety.md defines safety enforcement; docs/memory/allocation.md defines allocation; and docs/memory/shared-memory.md defines multi-participant shared access.

The requested docs/memory/virtual-memory.md is currently only a placeholder, so this is the correct place to establish the authoritative virtual-memory contract, without duplicating those other documents.

docs/memory/virtual-memory.md

# ULABI Virtual Memory Specification

Universal Language Application Binary Interface

**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Repository:** `Benwellonedge28/ULABI`  
**Document:** `docs/memory/virtual-memory.md`  
**Specification Area:** Memory / Virtual Memory  
**Status:** Architecture / Design Specification  
**Specification Version:** 0.2.0-draft  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI Virtual Memory contract.

The purpose of this specification is to establish language-neutral, runtime-neutral, operating-system-neutral, architecture-neutral, and implementation-independent semantics for virtual address spaces and virtual memory resources exposed through ULABI.

ULABI virtual memory provides a portable semantic model for systems that use virtual addressing while allowing implementations to use fundamentally different underlying mechanisms.

An implementation may use:

- hardware page tables;
- software-managed virtual memory;
- operating-system virtual memory;
- memory-management units;
- segmented address spaces;
- capability-based address spaces;
- managed runtime heaps;
- memory mappings;
- virtual machines;
- containers;
- device virtual memory;
- unified virtual addressing;
- sparse address spaces;
- demand paging;
- another implementation mechanism.

ULABI defines the interoperability contract.

It does not define one universal virtual-memory implementation.

---

# 2. Relationship to the ULABI Architecture

`ULABI-DESIGN.md` establishes:

- language neutrality;
- runtime neutrality;
- platform neutrality;
- architecture neutrality;
- implementation independence;
- explicit ownership;
- explicit lifetime;
- safe memory interoperability;
- zero-copy interoperability where safe;
- process isolation;
- capability security;
- distributed interoperability;
- deterministic behavior;
- profile-based extensions.

The ULABI ABI memory model establishes the general concept of a memory resource and address-space identity.

This document specializes those concepts for virtual memory.

The relationship is:

```text
                         ULABI
                           |
                    Memory Contract
                           |
                    Memory Model
                           |
                  Virtual Memory Model
                           |
        +------------------+------------------+
        |                  |                  |
   Address Space       Virtual Region      Mapping
        |                  |                  |
        +------------------+------------------+
                           |
                 Protection / Access
                           |
                    Physical / Device
                    Backing Resources

This document does not redefine:

general ownership;

general lifetime;

allocation;

general memory safety;

shared-memory synchronization;

general capability authorization;

serialization.


Those concerns remain defined by their respective specifications.


---

3. Non-Goals

This specification does not:

1. mandate paging;


2. mandate a page size;


3. mandate an MMU;


4. mandate an operating system;


5. mandate physical memory;


6. mandate demand paging;


7. mandate swapping;


8. mandate a particular virtual-address width;


9. mandate a particular pointer width;


10. expose physical addresses as portable ULABI addresses;


11. make virtual addresses portable between processes;


12. make virtual addresses portable between machines;


13. require every ULABI implementation to expose virtual memory;


14. define one universal page-table format;


15. define one universal kernel memory manager;


16. replace operating-system virtual-memory APIs;


17. make remote memory indistinguishable from local memory.




---

4. Normative Language

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

5. Fundamental Principle

> A ULABI virtual address is meaningful only within its declared address-space context.



A virtual address MUST NOT automatically be interpreted as a portable memory-resource identity.

For example:

Process A
virtual address: 0x100000

Process B
virtual address: 0x100000

These addresses may refer to completely different resources.

Therefore:

VirtualAddress != ResourceIdentity

and:

VirtualAddress != PortablePointer

unless an explicit ULABI contract establishes such semantics.


---

6. Address Space

A ULABI address space is an abstract namespace in which virtual addresses are interpreted.

Conceptually:

AddressSpace {
    address_space_id
    address_width
    regions
    mappings
    protection_policy
    capabilities
    locality
}

The physical representation is implementation-defined.

An address space MAY belong to:

a thread;

a process;

a runtime;

a virtual machine;

a container;

a device;

an accelerator;

another execution domain.


The scope MUST be explicit.


---

7. Address-Space Identity

Every virtual address used in a ULABI contract MUST be interpreted relative to an address-space identity.

Conceptually:

Address {
    address_space_id
    virtual_address
}

An implementation MAY use a more compact internal representation.

However, it MUST preserve the semantic distinction between:

address space A + address X

and:

address space B + address X

even when X has the same numerical value.


---

8. Address-Space Isolation

Separate address spaces SHOULD provide isolation between unrelated execution domains.

Conceptually:

+-----------------------+
| Address Space A       |
|                       |
|  Region A             |
+-----------------------+

          X

+-----------------------+
| Address Space B       |
|                       |
|  Region B             |
+-----------------------+

An operation in one address space MUST NOT gain access to another address space merely by constructing or guessing a numerical address.

Access across address spaces MUST require an explicitly defined mechanism.


---

9. Virtual Address

A virtual address is an implementation-relative address within an address space.

A virtual address MUST have:

an applicable address-space context;

a defined width or representational range;

defined validity semantics;

defined alignment semantics where applicable.


The numerical value of a virtual address has no universal meaning outside its address-space contract.


---

10. Address Width

An implementation MAY support different virtual-address widths.

Examples include:

32-bit
48-bit
52-bit
57-bit
64-bit
future widths

ULABI MUST NOT assume that the host architecture's native pointer width is the ULABI virtual-address width.

Where a portable virtual address is exposed, its width MUST be explicitly declared.


---

11. Canonical Address Representation

Where virtual addresses are serialized, transferred, logged, inspected, or otherwise represented outside their native address space, the representation MUST be explicitly defined.

A canonical representation MUST specify:

address width;

byte order;

validity range;

reserved bits;

canonicalization rules;

address-space identity.


Reserved or invalid address bits MUST NOT silently acquire semantic meaning.


---

12. Address Validity

A virtual address is valid only when all applicable conditions hold.

Conceptually:

ValidAddress =
    AddressSpaceExists
    AND AddressWithinRange
    AND RegionExists
    AND MappingValid
    AND PermissionAllowsAccess
    AND LifetimeValid
    AND CapabilityValid

A numerical address alone does not establish validity.


---

13. Virtual Memory Region

A virtual-memory region is a bounded portion of an address space.

Conceptually:

VirtualRegion {
    region_id
    address_space_id
    base
    size
    permissions
    mapping
    lifetime
    capabilities
}

A region MUST have authoritative bounds.

The implementation MUST prevent a region from being interpreted as extending beyond its declared extent.


---

14. Region Identity

A virtual-memory region MUST have a semantic identity distinct from its numerical base address.

This prevents problems when a region is:

unmapped;

remapped;

relocated;

resized;

replaced;

recreated.


A virtual address MAY change while the semantic resource remains the same.

Conversely, the same numerical address MAY later identify a different region.


---

15. Region Generations

Implementations SHOULD support generation tracking where address reuse can create stale references.

Example:

Region ID: 42
Generation: 7

After destruction and reuse:

Region ID: 42
Generation: 8

A stale reference to generation 7 MUST NOT gain access to generation 8.

Generation tracking MAY be implemented using:

generation counters;

protected handles;

capabilities;

resource tables;

another equivalent mechanism.



---

16. Mapping

A mapping establishes a relationship between a virtual region and a backing memory resource.

Conceptually:

Virtual Region
      |
      | mapping
      v
Backing Resource

The backing resource MAY be:

physical memory;

allocated memory;

shared memory;

persistent memory;

device memory;

file-backed memory;

copy-on-write storage;

another memory resource.


The mapping mechanism is implementation-defined.


---

17. Mapping Identity

A mapping MUST be distinguishable from the underlying backing resource.

One backing resource MAY have multiple mappings:

Backing Resource
       |
 +-----+-----+
 |     |     |
Map A Map B Map C
 |     |     |
A     B     C

Destroying one mapping MUST NOT automatically imply destruction of the backing resource unless the applicable resource contract explicitly requires it.


---

18. Mapping Permissions

A mapping MUST have explicit access permissions where permissions affect safety or security.

Possible permissions include:

None
Read
Write
Execute
ReadWrite
ReadExecute
WriteExecute
ReadWriteExecute
Atomic
DeviceAccess
DMA

Implementations MAY support only a subset.

An implementation MUST NOT silently grant stronger permissions than the contract authorizes.


---

19. Write-Xor-Execute

Writable and executable mappings SHOULD NOT be the default.

Where the platform supports write-xor-execute enforcement, an implementation SHOULD prohibit:

Writable + Executable

unless explicitly authorized by the applicable profile.

A request requiring executable writable memory MUST NOT silently downgrade its security semantics.


---

20. Read-Only Mappings

A read-only mapping MUST prevent unauthorized mutation through that mapping.

The following transformation MUST NOT silently increase authority:

ReadOnly Mapping
       |
       X
       v
Writable Mapping

Changing permissions requires an explicit authorized operation.


---

21. Execute Permissions

Executable mappings MUST require explicit authorization where execution permissions are security-sensitive.

ULABI implementations SHOULD support policies that distinguish:

Readable
Writable
Executable

rather than treating executable memory as implicitly readable or writable.


---

22. Mapping Creation

Mapping creation MUST validate:

address-space identity;

region validity;

backing-resource validity;

offset;

length;

alignment;

permissions;

capability;

lifetime;

requested address where applicable.


A failed mapping operation MUST NOT create a partially valid mapping.


---

23. Fixed-Address Mapping

An implementation MAY support requests for a specific virtual address.

Such requests MUST NOT be assumed portable.

A fixed-address mapping is valid only when the applicable address-space contract guarantees that address.

Example:

map(resource, address = 0x400000)

does not imply that another process or machine can map the same resource at 0x400000.


---

24. Relocatable Mapping

Portable ULABI interfaces SHOULD prefer relocatable mappings.

A relocatable mapping allows the implementation to choose an appropriate virtual address.

The semantic identity of the backing resource MUST remain independent of the selected address.


---

25. Unmapping

Unmapping removes a mapping from an address space.

Conceptually:

Address Space
      |
      | unmap
      v
Mapping Removed

Unmapping MUST NOT automatically destroy the backing resource unless the resource contract explicitly specifies that behavior.

After unmapping, references relying on that mapping MUST no longer use it.


---

26. Unmapping and Lifetime

A mapping and its backing resource have potentially different lifetimes.

For example:

Resource lifetime
+----------------------------------------+

Mapping A
   +-------------+

Mapping B
       +----------------+

The backing resource MAY survive after one mapping is removed.

The lifetime rules MUST be defined by the applicable resource contract.


---

27. Protection Changes

Changing mapping permissions MUST be an explicit operation.

Example:

Read
 |
 | authorized protection change
 v
ReadWrite

The operation MUST verify:

mapping identity;

requested permissions;

capability;

policy;

address-space authority.


An implementation MUST NOT silently grant stronger access.


---

28. Memory Mapping and Ownership

Mapping does not automatically transfer ownership.

For example:

Owner A
   |
   | map
   v
Process B

Process B may receive access without becoming the owner.

Ownership transfer requires an explicit ownership operation defined by the ownership contract.


---

29. Memory Mapping and Borrowing

A mapping MAY represent borrowed access.

The borrow MUST have:

owner;

borrower;

permitted operations;

lifetime;

validity conditions.


The mapping MUST NOT outlive its permitted borrowing contract.


---

30. Shared Memory Mapping

A shared-memory resource MAY be mapped into multiple address spaces.

This specification delegates shared-access semantics to:

docs/memory/shared-memory.md

The existence of multiple mappings MUST NOT imply unrestricted shared mutation.

Synchronization, visibility, consistency, and participant rights remain governed by the shared-memory contract.


---

31. Copy-on-Write

ULABI MAY support copy-on-write mappings.

Conceptually:

Resource
   |
   +---- Mapping A
   |
   +---- Mapping B

Initially both mappings MAY reference the same backing data.

After a write:

Mapping A
   |
   +---- Private Copy

Mapping B
   |
   +---- Original

The implementation MUST preserve the declared semantic behavior.

A write MUST NOT unexpectedly modify another participant's logically independent data.


---

32. Demand Paging

An implementation MAY use demand paging.

A page or region MAY initially have no resident physical backing while remaining logically valid according to the virtual-memory contract.

Page residency MUST remain an implementation detail unless a profile explicitly exposes it.


---

33. Page Faults

If an implementation uses page faults, page faults MUST NOT automatically become undefined ULABI behavior.

Where faults are observable through an applicable profile, the implementation MUST define:

fault classification;

recoverability;

error reporting;

retry behavior;

termination behavior.


Normal page-in operations SHOULD remain transparent to applications when they do not affect the semantic contract.


---

34. Paging and Determinism

The presence or absence of physical residency MUST NOT change the semantic result of a conforming operation.

For example:

resident page

and:

non-resident page

must produce equivalent semantic results when the operation is otherwise valid.

Performance differences MAY exist.

Semantic differences MUST be explicitly specified.


---

35. Swapping

An implementation MAY move memory between physical storage locations.

Examples:

RAM;

secondary storage;

compressed memory;

persistent memory;

another backing store.


Such movement MUST NOT change the semantic identity of a valid virtual resource.


---

36. Memory Remapping

A virtual region MAY be remapped to a different backing resource only through an explicit operation.

Remapping MUST define:

old mapping state;

new mapping state;

permissions;

synchronization;

lifetime;

invalidation behavior;

failure behavior.


A failed remapping operation MUST NOT leave the address space in an undefined intermediate state.


---

37. Atomic Remapping

Where supported, remapping operations SHOULD be atomic from the perspective of participating ULABI operations.

Conceptually:

Old Mapping
     |
  transition
     |
     v
New Mapping

Observers MUST NOT observe an undefined mixture of the old and new mapping states.


---

38. Region Resizing

Virtual regions MAY be resizable where the implementation supports it.

Resizing MUST explicitly define:

whether the base address changes;

whether existing mappings remain valid;

whether references are invalidated;

whether permissions change;

whether backing storage changes;

whether synchronization is required.


A stale reference MUST NOT silently access newly allocated or unrelated memory.


---

39. Guard Regions

Implementations SHOULD support guard regions where practical.

A guard region is an intentionally inaccessible region used to detect invalid access.

Example:

+------------------+
| Valid Region     |
+------------------+
| Guard Region     |
| NO ACCESS        |
+------------------+

Guard regions are an implementation mechanism for improving safety and diagnostics.

They MUST NOT be required for basic ULABI compatibility.


---

40. Null and Unmapped Addresses

ULABI MUST distinguish:

null;

unmapped;

invalid;

inaccessible;

released;

revoked;

absent.


These states MUST NOT be silently conflated.

A numerical zero address MUST NOT automatically mean every one of these states.


---

41. Reserved Address Ranges

An implementation MAY reserve portions of its address space.

Reserved ranges MUST NOT be treated as valid application memory unless explicitly mapped by the applicable contract.

Implementations MAY use reserved regions for:

guard pages;

runtime metadata;

kernel isolation;

security boundaries;

device mappings;

implementation-specific purposes.



---

42. Address-Space Layout

ULABI MUST NOT require a universal address-space layout.

For example:

Code
Heap
Stack
Shared Libraries
Mapped Files
Shared Memory
Device Memory

may appear at different addresses or in different arrangements on different implementations.

Only explicitly standardized layout requirements are portable.


---

43. Stack and Heap Separation

ULABI does not require a universal distinction between stack and heap.

Where an implementation exposes stack-backed memory across a ULABI boundary, its lifetime MUST be explicitly defined.

A call-scoped stack buffer MUST NOT be retained after its permitted lifetime merely because its address remains numerically valid.


---

44. Memory Domains

Virtual-memory mappings MAY identify different memory domains.

Examples:

HostMemory
DeviceMemory
SharedMemory
PersistentMemory
MappedFile
AcceleratorMemory
SecureMemory
RemoteMemory

A virtual address MUST NOT automatically imply that the underlying resource belongs to ordinary host memory.


---

45. Device Virtual Memory

ULABI MAY support device-specific virtual address spaces.

Examples include:

GPU virtual memory;

NPU virtual memory;

accelerator address spaces;

DMA address spaces.


Device addresses MUST remain distinct from host virtual addresses unless an explicit unified-addressing contract exists.


---

46. Unified Virtual Addressing

An implementation MAY provide unified virtual addressing.

However:

same address space

does not automatically imply:

same execution semantics

The contract MUST still define:

ownership;

permissions;

accessibility;

synchronization;

locality;

lifetime;

device semantics.



---

47. DMA and External Access

Virtual-memory mappings used for DMA or external devices MUST explicitly declare the applicable device-access semantics.

A normal CPU mapping MUST NOT automatically imply:

DMA capable

or:

Device accessible

These permissions require explicit authorization.


---

48. Security

Virtual-memory operations MUST integrate with the ULABI security model.

At minimum, security-sensitive operations MUST consider:

capability;

authority;

address-space identity;

resource identity;

permissions;

isolation;

lifetime;

revocation.


A component MUST NOT gain access to another component's address space merely by knowing an address.


---

49. Capability Security

A virtual-memory operation MAY require a capability.

Examples:

Map
Unmap
Read
Write
Execute
Resize
Remap
Share
Revoke

Capabilities MUST be checked before performing protected operations.

A retained numerical address MUST NOT substitute for a required capability.


---

50. Revocation

If authority to a mapping or resource is revoked:

Valid
  |
revoke
  |
  v
Revoked

future access through that authority MUST fail according to the applicable contract.

Cached pointers, mappings, handles, or language objects MUST NOT silently bypass revocation.


---

51. Isolation

ULABI implementations SHOULD use hardware or software isolation mechanisms where available.

Possible mechanisms include:

separate page tables;

memory protection;

capability-based addressing;

sandboxing;

virtual machines;

protected processes;

hardware domains.


The implementation mechanism is not normative.

The isolation guarantee is.


---

52. Cross-Process Virtual Memory

A virtual address from Process A MUST NOT automatically be valid in Process B.

Cross-process memory access requires an explicit mechanism such as:

shared memory;

memory transfer;

handle-based access;

capability-based mapping;

IPC-backed resource;

another standardized mechanism.



---

53. Cross-Machine Virtual Memory

A virtual address MUST NOT automatically be interpreted on another machine.

Distributed memory abstractions MUST use explicit distributed-memory semantics.

A remote resource MUST remain distinguishable from an ordinary local pointer unless a specific profile establishes otherwise.


---

54. Virtual Memory and Serialization

Virtual addresses MUST NOT normally be serialized as portable memory references.

For serialization, implementations SHOULD use:

resource identifiers;

handles;

canonical descriptors;

serialized values;

explicit shared-memory descriptors.


A serialized raw pointer MUST NOT be treated as a portable ULABI pointer.


---

55. Virtual Memory and Zero-Copy

Virtual memory may enable zero-copy interoperability.

However, zero-copy is valid only when all applicable contracts establish:

compatible representation;

ownership;

lifetime;

bounds;

permissions;

address-space validity;

synchronization;

capability authorization.


The existence of a mapping alone does not make zero-copy safe.


---

56. Virtual Memory and Memory Safety

This specification relies on:

docs/memory/memory-safety.md

The following MUST remain enforced:

bounds;

alignment;

lifetime;

ownership;

capability;

type validity;

initialization;

access permissions;

stale-resource detection.


Virtual memory MUST NOT be used to bypass memory-safety requirements.


---

57. Virtual Memory and Allocation

This specification relies on:

docs/memory/allocation.md

Allocation creates or reserves memory resources.

Virtual mapping determines how those resources are represented within an address space.

The two operations MUST NOT be silently conflated.

For example:

allocate(resource)
        |
        v
Memory Resource
        |
        | map
        v
Virtual Region


---

58. Virtual Memory and Ownership

This specification relies on:

docs/memory/ownership.md

Mapping does not imply ownership transfer.

Unmapping does not necessarily imply resource destruction.

Ownership changes MUST use the ownership contract.


---

59. Virtual Memory and Lifetimes

This specification relies on:

docs/memory/lifetimes.md

A mapping MUST remain valid only for its declared lifetime.

If the backing resource expires or is released, dependent mappings MUST be invalidated or otherwise handled according to the resource contract.


---

60. Virtual Memory and Shared Memory

This specification relies on:

docs/memory/shared-memory.md

Shared mappings MUST preserve the shared-memory contract concerning:

participants;

rights;

synchronization;

visibility;

consistency;

lifetime;

revocation.



---

61. Virtual Memory and ABI Calling Conventions

A function parameter MUST NOT expose an implementation-specific virtual address as a portable ULABI pointer unless the calling convention explicitly defines that representation.

For cross-language calls, portable memory should normally be represented using:

values;

descriptors;

handles;

bounded buffers;

views;

capabilities.



---

62. Virtual Memory Descriptors

ULABI implementations SHOULD provide a canonical virtual-memory descriptor.

Conceptually:

VirtualMemoryDescriptor {
    address_space_id
    region_id
    generation
    base
    length
    permissions
    memory_domain
    lifetime
    capabilities
}

The exact binary encoding belongs to the relevant schema/ABI specification.


---

63. Descriptor Validation

A descriptor MUST be validated before being used to establish or access a mapping.

Validation SHOULD include:

identity
generation
address-space
bounds
permissions
lifetime
capability
domain

An invalid descriptor MUST NOT be interpreted as valid memory.


---

64. Stale Descriptors

A descriptor referring to an old region generation MUST NOT access a newly created region that happens to reuse the same numerical address.

Example:

Old:
region=42 generation=7 address=0x100000

New:
region=42 generation=8 address=0x100000

The old descriptor MUST remain invalid.


---

65. Mapping Failure

Mapping failures MUST be explicit.

Possible failure categories include:

InvalidAddressSpace

InvalidRegion

InvalidResource

InvalidRange

AddressConflict

AlignmentError

PermissionDenied

CapabilityDenied

AddressSpaceExhausted

ResourceUnavailable

UnsupportedMapping

UnsupportedPermission

LifetimeExpired

StaleGeneration

SecurityPolicyViolation

DeviceUnavailable


Implementations MAY extend this taxonomy.


---

66. Failure Atomicity

A failed mapping operation MUST NOT silently create a partially valid mapping.

Unless explicitly specified otherwise:

Before:
Old valid state

Attempt mapping

Failure:
Old valid state remains

The implementation MUST preserve a valid previous state where the operation contract requires failure atomicity.


---

67. Recovery

Virtual-memory recovery MUST be bounded and policy-controlled.

Permitted recovery mechanisms MAY include:

retrying a failed mapping;

remapping an equivalent resource;

restoring a previous mapping;

invalidating stale mappings;

quarantining a failed region;

escalating the failure.


An implementation MUST NOT arbitrarily modify unrelated mappings in an attempt to recover.


---

68. Deterministic Failure

A conforming implementation MUST expose failures through defined ULABI error semantics.

The implementation MUST NOT turn a mapping failure into apparently valid memory access.


---

69. Concurrency

Concurrent mapping operations MUST have explicitly defined semantics.

If two operations modify the same virtual region concurrently, the implementation MUST define whether:

operations are serialized;

operations conflict;

one operation wins;

both fail;

transactions are used.


Undefined concurrent mapping behavior MUST NOT be part of the portable contract.


---

70. Quotas

Virtual-memory implementations MAY impose:

address-space quotas;

mapping-count quotas;

mapped-byte quotas;

device-memory quotas;

permission-specific quotas.


Quota failures MUST be explicit.

They MUST NOT corrupt existing valid mappings.


---

71. Resource Accounting

Where resource accounting is exposed, implementations SHOULD be able to distinguish:

Virtual Address Space
Mapped Virtual Memory
Resident Physical Memory
Backing Resource
Shared Mapping
Device Mapping

These quantities MUST NOT be silently treated as equivalent.


---

72. Portability

A portable ULABI component MUST NOT assume:

a particular virtual-address value;

a particular page size;

a particular address-space layout;

a particular pointer width;

a particular page-table implementation;

a particular paging policy;

a particular physical-memory layout.


Such assumptions require an explicit platform or architecture profile.


---

73. Platform Profiles

Platform-specific virtual-memory requirements MAY be defined through profiles.

Possible profiles include:

ULABI POSIX Memory Profile;

ULABI Windows Memory Profile;

ULABI Embedded Memory Profile;

ULABI WebAssembly Memory Profile;

ULABI GPU Memory Profile;

ULABI Accelerator Memory Profile;

ULABI Real-Time Memory Profile;

ULABI Safety-Critical Memory Profile.


Profiles MUST NOT alter the meaning of the ULABI Core without an explicit versioned extension.


---

74. Embedded Systems

An embedded implementation MAY have:

no MMU;

limited virtual addressing;

static mappings;

MPU-based protection;

physically addressed memory.


Such systems MAY implement the ULABI virtual-memory profile only to the extent applicable.

Absence of hardware virtual memory MUST NOT prevent participation in ULABI generally.


---

75. WebAssembly

A WebAssembly implementation MAY expose its linear memory through a ULABI adapter.

The WebAssembly linear-memory index MUST NOT automatically be treated as a host-process virtual address.

The adapter MUST preserve:

bounds;

lifetime;

ownership;

type;

permissions;

address-space semantics.



---

76. Safety-Critical Systems

Safety-critical profiles SHOULD require:

explicit memory regions;

bounded mappings;

deterministic failure;

static or verified mapping policies where applicable;

no silent permission escalation;

explicit recovery;

traceable configuration.


The generic ULABI virtual-memory model MUST remain independent of any one safety standard.


---

77. Real-Time Systems

Real-time profiles MAY prohibit or constrain:

demand paging;

blocking mapping operations;

unpredictable allocation;

dynamic remapping;

page faults during critical execution.


Such constraints belong to the profile rather than the universal Core.


---

78. Debugging and Diagnostics

Implementations SHOULD expose diagnostics sufficient to identify:

address-space identity;

region identity;

generation;

mapping;

permissions;

lifetime;

capability failures;

address conflicts;

stale references.


Diagnostics MUST NOT expose protected information merely because a memory fault occurred.


---

79. Observability

Where observability is enabled, virtual-memory events MAY include:

Map
Unmap
Protect
Remap
Resize
Fault
Invalidate
Revoke
Restore

Events SHOULD carry stable resource identifiers rather than relying solely on raw addresses.


---

80. Conformance Requirements

A conforming ULABI virtual-memory implementation MUST:

1. distinguish address-space identity from numerical address;


2. define virtual-region bounds;


3. enforce mapping permissions;


4. prevent unauthorized cross-address-space access;


5. define mapping lifetime;


6. define unmapping behavior;


7. prevent stale mappings from silently accessing replacement resources;


8. define mapping failures;


9. integrate with ULABI memory safety;


10. integrate with ownership;


11. integrate with lifetime;


12. integrate with allocation;


13. integrate with shared memory where applicable;


14. preserve capability restrictions;


15. avoid treating raw virtual addresses as universal resource identities.




---

81. Conformance Test Categories

The ULABI conformance suite MUST eventually test at least:

Address-space isolation

same numerical address in different spaces;

unauthorized cross-space access;

valid cross-space capability access.


Mapping

valid mapping;

invalid mapping;

permission enforcement;

alignment;

bounds;

address conflicts.


Unmapping

valid unmap;

double unmap;

use after unmap;

backing-resource lifetime.


Protection

read-only mapping;

unauthorized write;

permission escalation;

executable mapping policy.


Generation safety

stale region;

address reuse;

stale descriptor;

generation mismatch.


Lifetime

expired mapping;

released backing resource;

participant termination;

revocation.


Shared memory

multiple mappings;

synchronization;

permission isolation;

stale shared mappings.


Device memory

host/device distinction;

invalid device mapping;

unauthorized DMA access.


Failure atomicity

mapping failure;

remapping failure;

resizing failure;

permission-change failure.


Portability

different address widths;

different page sizes;

different address layouts;

relocatable mappings.



---

82. Reference Data Structures

A reference implementation MAY expose structures conceptually equivalent to:

AddressSpaceId
RegionId
MappingId
RegionGeneration
VirtualAddress
VirtualRange
MemoryPermissions
MemoryDomain
VirtualMemoryDescriptor
MappingContract

These are semantic concepts.

Their language-specific representation is implementation-defined.


---

83. Reference Interface

A reference ULABI memory interface MAY provide operations equivalent to:

create_address_space()
destroy_address_space()

create_region()
destroy_region()

map()
unmap()

protect()
remap()

resize_region()

query_region()
query_mapping()

validate_address()
validate_descriptor()

revoke_mapping()

These names are illustrative.

The normative interface must eventually be defined in the machine-readable ULABI interface schemas.


---

84. Required Invariants

The following invariants are fundamental:

Invariant 1 — Address-space isolation

A virtual address MUST NOT cross address-space boundaries without an explicit mechanism.

Invariant 2 — Resource identity

A numerical virtual address MUST NOT be treated as the sole resource identity.

Invariant 3 — Bounds

Every valid virtual region MUST have authoritative bounds.

Invariant 4 — Permissions

Access MUST NOT exceed the permissions of the mapping.

Invariant 5 — Lifetime

Expired mappings MUST NOT remain valid.

Invariant 6 — Generation

A stale mapping MUST NOT access a replacement resource.

Invariant 7 — Ownership

Mapping MUST NOT silently transfer ownership.

Invariant 8 — Capability

Protected operations MUST require appropriate authority.

Invariant 9 — Failure safety

Failed operations MUST NOT silently create invalid mappings.

Invariant 10 — Portability

Portable ULABI code MUST NOT depend on implementation-specific virtual addresses.


---

85. Security Requirements

A conforming implementation:

MUST prevent unauthorized address-space access.

MUST prevent unauthorized permission escalation.

MUST validate mapping capabilities.

MUST prevent stale mapping reuse.

MUST protect address-space identity.

MUST prevent invalid mappings from becoming valid through numerical address reuse.

SHOULD use hardware memory protection where available.

SHOULD use W^X where supported.

MUST NOT expose protected address-space information unnecessarily.


---

86. Integration Matrix

This document is authoritative for virtual-memory semantics.

It integrates with:

docs/abi/memory-model.md

general memory-resource abstraction;

address-space concepts;

memory layout;

pointer restrictions;

zero-copy boundaries.


docs/memory/memory-safety.md

bounds;

lifetime;

alignment;

initialization;

stale-resource safety;

authorization.


docs/memory/ownership.md

ownership state;

ownership transfer;

borrowed access;

release authority.


docs/memory/lifetimes.md

mapping lifetime;

resource lifetime;

expiration;

invalidation.


docs/memory/allocation.md

creation of backing memory resources;

allocation domains;

release;

allocation failure.


docs/memory/shared-memory.md

multiple mappings;

participant rights;

synchronization;

visibility;

consistency.


docs/security/security-model.md

authorization;

capability requirements;

isolation;

security boundaries.


docs/security/capability-security.md

mapping capabilities;

permission delegation;

revocation.


docs/compatibility/feature-negotiation.md

virtual-memory feature discovery;

optional capability negotiation.


docs/compatibility/capability-discovery.md

address-space and mapping capabilities.


docs/platforms/architectures.md

architecture-specific address-space constraints.


docs/platforms/operating-systems.md

operating-system mapping profiles.


docs/hardware/cpu.md

CPU memory-management capabilities.


docs/hardware/gpu.md

GPU virtual-memory profiles.


docs/hardware/npu.md

accelerator virtual-memory profiles.


docs/hardware/fpga.md

device-memory mappings where applicable.


docs/tooling/debugger-interface.md

virtual-memory diagnostics.


docs/observability/diagnostics.md

mapping and fault diagnostics.


docs/standards/conformance.md

conformance requirements.


docs/standards/test-suite.md

executable virtual-memory tests.



---

87. Source-of-Truth Boundaries

To prevent future document duplication:

Concern	Authoritative document

General ABI memory boundary	docs/abi/memory-model.md
Allocation	docs/memory/allocation.md
Ownership	docs/memory/ownership.md
Lifetimes	docs/memory/lifetimes.md
General memory safety	docs/memory/memory-safety.md
Virtual memory	this document
Shared memory	docs/memory/shared-memory.md
Serialization	docs/distributed/serialization.md
Capabilities	docs/security/capability-security.md
Security model	docs/security/security-model.md
Compatibility negotiation	docs/compatibility/feature-negotiation.md


No other document should redefine the normative virtual-memory contract.


---

88. Implementation Independence

This specification MUST be implementable by systems using fundamentally different memory architectures.

Examples include:

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

WebAssembly;

embedded runtimes;

operating-system kernels;

virtual machines;

accelerator runtimes;

future programming languages.


ULABI MUST remain independent of all of them.

Zamani and Sankofa may independently implement or target this contract, but neither language defines this specification.


---

89. Future Extensions

Future profiles MAY define:

huge pages;

transparent huge pages;

compressed memory;

persistent memory;

encrypted memory;

confidential memory;

memory tagging;

capability hardware;

NUMA-aware virtual memory;

transactional mappings;

heterogeneous memory;

CXL memory;

accelerator virtual memory;

secure enclaves;

verified address spaces;

real-time virtual memory;

formally verified mappings.


These MUST be introduced as versioned extensions or profiles rather than silently changing the Core contract.


---

90. Final Principle

> Virtual memory is an implementation mechanism for representing memory resources inside an address space; it is not itself a universal memory-resource identity.



ULABI therefore standardizes:

Address Space
      |
Virtual Region
      |
Mapping
      |
Backing Resource
      |
Permissions
      |
Lifetime
      |
Capability
      |
Safety

while deliberately leaving the underlying implementation to the participating system.

A conforming implementation is free to use any internal virtual-memory architecture provided that the externally observable ULABI contract remains compatible, safe, explicit, and testable.

## What this means architecturally

The important thing is that I **did not make `virtual-memory.md` another generic memory document**. The existing `memory-model.md` already owns the fundamental memory abstraction, while `memory-safety.md`, `allocation.md`, and `shared-memory.md` already own their respective areas. That separation is visible in the repository itself.   

The new document therefore owns one thing:

> **How virtual address spaces, regions, mappings, permissions, address reuse, and virtual-memory-specific failures behave at a ULABI boundary.**

---

# Required file/module plan

The repository is currently overwhelmingly specification/documentation-oriented; the repository tree already contains substantial ABI, architecture, compatibility, distributed, hardware, and memory specifications, while the requested virtual-memory document was still a placeholder. 

I recommend **not creating implementation code before the contracts are frozen**. The implementation should follow the specification layers.

## Phase 1 — independent semantic contracts

These should be completed first because they establish concepts other documents consume.

### Memory contracts

```text
docs/memory/
├── allocation.md
├── ownership.md
├── lifetimes.md
├── memory-safety.md
├── virtual-memory.md          ← current document
└── shared-memory.md

Responsibilities:

File	Owns

allocation.md	acquiring/resizing/releasing memory
ownership.md	who controls/release/transfers resources
lifetimes.md	validity intervals
memory-safety.md	safety invariants
virtual-memory.md	address spaces/mappings
shared-memory.md	multi-participant access


This creates a clean dependency chain:

Allocation
    ↓
Ownership
    ↓
Lifetime
    ↓
Memory Safety
    ↓
Virtual Memory
    ↓
Shared Memory

They should not redefine one another.


---

Phase 2 — ABI foundation

Existing documents that must ultimately consume the memory contracts:

docs/abi/
├── core-abi.md
├── calling-convention.md
├── data-types.md
├── memory-model.md
├── stack-model.md
├── register-model.md
├── exception-model.md
└── return-values.md

The key integration is:

core-abi
   |
   +── data-types
   |
   +── calling-convention
   |
   +── memory-model
            |
            +── allocation
            +── ownership
            +── lifetimes
            +── memory-safety
            +── virtual-memory
            +── shared-memory

The ABI documents should reference these memory specifications instead of duplicating their semantics.


---

Phase 3 — interoperability

Required modules:

docs/interoperability/
├── language-interoperability.md
├── foreign-function-interface.md
├── object-model.md
├── name-mangling.md
├── symbol-resolution.md
└── cross-language-data.md

Their memory integration is:

Language A
    |
Language Adapter
    |
ULABI FFI
    |
Memory Descriptor / Handle / Value
    |
Language B

They must never assume that a C pointer, Rust reference, Java object, Python object, or any other language-specific representation is automatically a ULABI memory reference.


---

Phase 4 — runtime contracts

docs/runtime/
├── runtime-interface.md
├── process-model.md
├── threading.md
├── async-model.md
├── concurrency.md
└── resource-management.md

Memory integration:

Runtime
 |
 +-- AddressSpaceManager
 +-- MemoryResourceManager
 +-- MappingManager
 +-- OwnershipManager
 +-- LifetimeManager
 +-- ResourceManager

Async operations are particularly important because a borrowed virtual-memory region cannot simply survive an asynchronous operation unless the lifetime contract explicitly allows it.


---

Phase 5 — security integration

docs/security/
├── security-model.md
├── capability-security.md
├── sandboxing.md
├── cryptography.md
├── authentication.md
├── authorization.md
├── zero-trust.md
├── secure-loading.md
└── supply-chain-security.md

The important modules are:

Capability
    ↓
AddressSpace Authorization
    ↓
Mapping Authorization
    ↓
Permission Enforcement
    ↓
Revocation

Virtual-memory permissions must therefore never become a second, competing security model.


---

Phase 6 — compatibility

docs/compatibility/
├── backwards-compatibility.md
├── forwards-compatibility.md
├── feature-negotiation.md
├── capability-discovery.md
└── graceful-degradation.md

These need to answer questions such as:

Does implementation A support:
    virtual memory?
    64-bit addresses?
    shared mappings?
    executable mappings?
    device mappings?
    huge pages?
    memory protection?

without forcing the implementation to pretend that unsupported features exist.


---

Phase 7 — distributed boundary

docs/distributed/
├── distributed-abi.md
├── remote-calls.md
├── serialization.md
├── service-discovery.md
├── distributed-errors.md
└── consensus-boundaries.md

The critical rule:

VirtualAddress
      ≠
RemoteAddress

A remote memory descriptor must therefore be explicitly represented as a distributed resource rather than serialized as a raw pointer.


---

Required machine-readable schemas

The documentation should eventually produce schemas under:

schemas/
├── memory/
│   ├── memory-resource.schema.json
│   ├── memory-descriptor.schema.json
│   ├── ownership.schema.json
│   ├── lifetime.schema.json
│   ├── allocation.schema.json
│   ├── virtual-memory.schema.json
│   ├── address-space.schema.json
│   ├── virtual-region.schema.json
│   ├── mapping.schema.json
│   ├── mapping-permissions.schema.json
│   └── shared-memory.schema.json
│
├── errors/
│   ├── memory-errors.schema.json
│   ├── mapping-errors.schema.json
│   └── resource-errors.schema.json
│
└── compatibility/
    ├── memory-capabilities.schema.json
    └── feature-negotiation.schema.json

These schemas should be generated from the normative contracts, not invented independently of them.


---

Required code modules

Since ULABI should be implementation-independent, I would eventually structure the reference implementation around semantic modules rather than around one programming language.

implementations/
└── reference/
    ├── core/
    │   ├── interface_id
    │   ├── version
    │   ├── type_id
    │   └── error
    │
    ├── memory/
    │   ├── resource
    │   ├── allocation
    │   ├── ownership
    │   ├── lifetime
    │   ├── safety
    │   ├── virtual_memory
    │   ├── address_space
    │   ├── region
    │   ├── mapping
    │   ├── permissions
    │   ├── shared_memory
    │   ├── handle
    │   └── generation
    │
    ├── types/
    │   ├── primitive
    │   ├── record
    │   ├── variant
    │   ├── option
    │   └── result
    │
    ├── abi/
    │   ├── calling_convention
    │   ├── marshalling
    │   ├── descriptors
    │   └── invocation
    │
    ├── security/
    │   ├── capability
    │   ├── authorization
    │   ├── isolation
    │   └── revocation
    │
    ├── compatibility/
    │   ├── feature_negotiation
    │   ├── capability_discovery
    │   └── compatibility
    │
    └── diagnostics/
        ├── validation
        ├── errors
        └── tracing

Virtual-memory implementation modules specifically

The virtual-memory contract eventually maps cleanly to:

memory/
├── address_space
├── virtual_address
├── virtual_region
├── region_generation
├── mapping
├── mapping_permissions
├── mapping_table
├── protection
├── remapping
├── region_resize
├── address_validation
├── descriptor_validation
├── mapping_capability
├── mapping_revocation
├── page_fault
├── guard_region
└── virtual_memory_errors

These are semantic modules, not mandates that every implementation must expose files with exactly these names.


---

Conformance code

The eventual test structure should be:

tests/
├── memory/
│   ├── allocation/
│   ├── ownership/
│   ├── lifetime/
│   ├── safety/
│   ├── virtual_memory/
│   │   ├── address_space_isolation
│   │   ├── address_validation
│   │   ├── region_bounds
│   │   ├── mapping
│   │   ├── unmapping
│   │   ├── permissions
│   │   ├── protection
│   │   ├── remapping
│   │   ├── resizing
│   │   ├── generation_safety
│   │   ├── stale_mapping
│   │   ├── revocation
│   │   ├── failure_atomicity
│   │   └── device_memory
│   │
│   └── shared_memory/
│
├── abi/
├── interoperability/
├── security/
├── compatibility/
└── distributed/

Then:

conformance/
├── core/
├── types/
├── memory/
│   ├── allocation/
│   ├── ownership/
│   ├── lifetime/
│   ├── safety/
│   ├── virtual-memory/
│   └── shared-memory/
├── ffi/
├── security/
├── async/
├── distributed/
└── profiles/

The result should eventually permit a report like:

ULABI Core                 PASS
ULABI Types                PASS
ULABI ABI                  PASS
ULABI Allocation           PASS
ULABI Ownership            PASS
ULABI Lifetime             PASS
ULABI Memory Safety        PASS
ULABI Virtual Memory       PASS
ULABI Shared Memory        PASS
ULABI Security             PASS
ULABI FFI                  PASS
ULABI Compatibility        PASS

rather than a vague "ULABI compatible" claim.


---

Final dependency architecture

The cleanest overall dependency graph is:

ULABI-DESIGN.md
                       |
                 ULABI-SPEC.md
                       |
              +--------+--------+
              |                 |
          Core ABI         Universal Types
              |                 |
              +--------+--------+
                       |
                 Memory Model
                       |
          +------------+------------+
          |            |            |
     Allocation     Ownership    Lifetime
          |            |            |
          +------------+------------+
                       |
                 Memory Safety
                       |
          +------------+------------+
          |                         |
   Virtual Memory             Shared Memory
          |                         |
          +------------+------------+
                       |
                 FFI / Runtime
                       |
              Security / Capability
                       |
                Compatibility
                       |
                Distributed ABI
                       |
              Conformance Suite
                       |
            Independent Implementations

This is the important architectural result: virtual-memory.md can now be considered complete as a standalone contract. Future documents should integrate with the interfaces and source-of-truth boundaries already declared here, rather than coming back and rewriting this document.

And, importantly, nothing in this specification makes ULABI depend on Zamani or Sankofa; they remain independent possible implementations/targets of the ULABI contract, exactly as required by the master architecture.