I inspected the actual Benwellonedge28/ULABI repository before drafting this. The repository already has a substantial ULABI-DESIGN.md, a substantial docs/abi/memory-model.md, a substantial docs/memory/allocation.md, and a substantial docs/memory/lifetimes.md.

Importantly, docs/memory/memory-safety.md is currently only a placeholder, as are ownership.md and shared-memory.md/virtual-memory.md at the current repository state.

I therefore designed memory-safety.md as the authoritative safety specification, rather than duplicating allocation, ownership, lifetime, virtual-memory, or shared-memory semantics.

The repository architecture already explicitly requires language/runtime/platform neutrality, safe memory interoperability, explicit ownership/lifetime boundaries, capability security, zero-copy where safe, deterministic behavior, and implementation independence.

docs/memory/memory-safety.md

ULABI Memory Safety Specification

Universal Language Application Binary Interface

Project: ULABI
Full Name: Universal Language Application Binary Interface
Repository: "Benwellonedge28/ULABI"
Document: "docs/memory/memory-safety.md"
Specification Area: Memory / Memory Safety
Status: Architecture / Design Specification
Specification Version: 0.2.0-draft
License: Apache-2.0

---

1. Purpose

This document defines the ULABI memory-safety contract.

The purpose of this specification is to establish language-neutral, runtime-neutral, platform-neutral and implementation-independent requirements that prevent, detect, contain and report invalid memory behavior at ULABI interoperability boundaries.

ULABI memory safety exists to ensure that a component implemented in one language or runtime cannot cause another component to observe invalid or unauthorized memory behavior merely because their internal memory-management models differ.

ULABI memory safety applies to:

- values;
- references;
- pointers;
- handles;
- buffers;
- arrays;
- records;
- views;
- shared memory;
- transferred memory;
- borrowed memory;
- asynchronous operations;
- accelerator memory;
- mapped memory;
- process-local memory;
- cross-process memory;
- distributed memory descriptors;
- memory-backed resources.

ULABI does not prescribe a single memory-management strategy.

An implementation may use:

- manual memory management;
- ownership and borrowing;
- reference counting;
- tracing garbage collection;
- arenas;
- regions;
- pools;
- immutable data;
- runtime-managed references;
- hardware memory protection;
- capability systems;
- static analysis;
- dynamic analysis;
- another implementation strategy.

ULABI defines the observable safety contract at the interoperability boundary.

---

2. Relationship to the ULABI Architecture

"ULABI-DESIGN.md" establishes ULABI as:

«The universal interoperability contract, not a universal programming language, compiler, operating system, runtime or memory-management system.»

The architecture already establishes:

- language neutrality;
- runtime neutrality;
- platform neutrality;
- architecture neutrality;
- vendor neutrality;
- implementation independence;
- explicit ownership;
- explicit lifetime;
- safe memory interoperability;
- zero-copy where safe;
- capability-based security;
- process isolation;
- deterministic behavior;
- failure-oriented design;
- conformance testing.

This specification specializes those principles for memory safety.

The relationship is:

                         ULABI
                           |
                    Memory Contract
                           |
                  +--------+--------+
                  |                 |
             Memory Model       Memory Safety
                  |                 |
          +-------+-------+         |
          |       |       |         |
      Ownership Lifetime Allocation |
          |       |       |         |
          +-------+-------+---------+
                           |
                    Safety Enforcement
                           |
              +------------+------------+
              |            |            |
          Validation   Isolation    Recovery
              |            |            |
              +------------+------------+
                           |
                    Implementation

This document therefore does not redefine:

- allocation;
- ownership;
- lifetime;
- virtual memory;
- shared-memory synchronization;
- serialization;
- capability authorization.

Instead, it defines the safety properties those mechanisms must satisfy when combined.

---

3. Normative Language

The terms below are normative:

- MUST
- MUST NOT
- REQUIRED
- SHALL
- SHALL NOT
- SHOULD
- SHOULD NOT
- MAY

A conforming implementation MUST satisfy all applicable MUST and MUST NOT requirements.

---

4. Fundamental Safety Principle

«A ULABI implementation MUST NOT allow an interoperability boundary to convert an invalid, unauthorized, out-of-bounds, expired, incompatible or otherwise unsafe memory operation into apparently valid behavior.»

Memory safety must be preserved across the boundary.

For example:

Language A
    |
    | invalid memory operation
    v
ULABI Boundary
    |
    X
    |
    v
Language B

The boundary MUST NOT silently transform the invalid operation into valid semantic data.

The implementation MUST instead:

1. reject the operation;
2. safely isolate it;
3. report a defined error;
4. terminate the offending operation when necessary; or
5. invoke an explicitly authorized recovery mechanism.

---

5. Safety Properties

A conforming ULABI implementation MUST define mechanisms for preventing or detecting the following classes of violation where applicable:

1. use-after-free;
2. use-after-release;
3. use-after-transfer;
4. use-after-expiration;
5. use-after-revocation;
6. double release;
7. double ownership transfer;
8. invalid pointer use;
9. stale handle use;
10. out-of-bounds access;
11. buffer overflow;
12. buffer underflow;
13. integer overflow affecting memory operations;
14. integer underflow affecting memory operations;
15. invalid alignment;
16. invalid type interpretation;
17. type confusion;
18. unauthorized mutation;
19. unauthorized sharing;
20. unauthorized release;
21. unauthorized memory mapping;
22. invalid memory-domain access;
23. invalid address-space access;
24. unsafe zero-copy access;
25. invalid concurrent access;
26. data races where the applicable profile requires their prevention;
27. lifetime violations;
28. invalid reallocation use;
29. stale views;
30. capability violations;
31. memory corruption crossing an isolation boundary.

The exact enforcement mechanism is implementation-defined.

The safety outcome is normative.

---

6. Memory Safety Boundary

Every ULABI memory boundary MUST establish sufficient metadata to determine the legal use of a resource.

At minimum, where applicable, the boundary MUST be able to establish:

resource identity
type
size
bounds
alignment
ownership
lifetime
permissions
locality
memory domain
sharing state
capabilities

Not every representation needs to physically encode every field.

However, the implementation MUST possess enough authoritative information to enforce the applicable contract.

---

7. Valid Memory

Memory is valid only when all required validity conditions hold.

Conceptually:

Valid =
    Exists
    AND WithinBounds
    AND CorrectType
    AND CorrectAlignment
    AND LifetimeValid
    AND OwnershipValid
    AND CapabilityValid
    AND AccessAuthorized
    AND MemoryDomainValid

If any mandatory condition fails, the operation MUST NOT be treated as valid.

---

8. Memory Validity Is Not Pointer Existence

The existence of a pointer-like value does not establish memory validity.

The following implication is forbidden:

pointer exists
     =>
memory is valid

Instead:

pointer/handle
      |
      v
validation
      |
 +----+----+
 |         |
valid    invalid
 |         |
 v         v
access    reject

This distinction is essential for compatibility with:

- garbage-collected languages;
- ownership-based languages;
- managed runtimes;
- capability systems;
- handles;
- shared memory;
- distributed descriptors.

---

9. Bounds Safety

Every bounded memory resource MUST have an authoritative logical extent.

For a resource:

base
length

an access range:

[offset, offset + access_size)

is valid only if:

0 <= offset
AND
access_size >= 0
AND
offset + access_size <= length

The addition MUST be overflow-checked.

An implementation MUST NOT perform the arithmetic using a representation that can wrap into a smaller value and thereby bypass bounds checking.

---

10. Integer Overflow Protection

Memory-related arithmetic MUST be checked before it is used for:

- allocation;
- indexing;
- slicing;
- offset calculation;
- stride calculation;
- array-size calculation;
- buffer-size calculation;
- copy-size calculation;
- alignment calculation;
- mapping-size calculation.

For example:

element_count * element_size

MUST be validated before it becomes an allocation size.

Overflow MUST result in a defined failure.

It MUST NOT result in an undersized allocation followed by apparently valid access.

---

11. Underflow Protection

Arithmetic involving:

- offsets;
- lengths;
- indexes;
- capacities;
- sizes;
- pointer-relative calculations;

MUST prevent unsigned or signed underflow from producing an invalid memory region that is subsequently treated as valid.

---

12. Out-of-Bounds Access

An implementation MUST prevent or detect:

- access before the beginning of a resource;
- access after the end of a resource;
- access beyond logical length;
- access beyond allocated capacity where the contract does not expose capacity;
- invalid multidimensional indexing;
- invalid stride traversal.

An out-of-bounds operation MUST NOT silently access another ULABI resource.

---

13. Buffer Safety

Buffers crossing ULABI boundaries MUST have explicit:

- size;
- capacity where exposed;
- element semantics where typed;
- alignment;
- ownership;
- lifetime;
- access permissions.

A buffer consumer MUST NOT assume that capacity equals logical length.

---

14. Array Safety

For an array:

Array<T>

the contract MUST define:

- element type;
- element size or semantic layout;
- element count;
- indexing rules;
- bounds;
- alignment;
- lifetime;
- ownership;
- mutation permissions.

Multidimensional arrays MUST additionally define:

- shape;
- stride;
- layout;
- offset semantics.

---

15. Slice and View Safety

A view or slice MUST NOT outlive the resource from which it was derived unless it owns or otherwise retains an authorized lifetime of that resource.

Conceptually:

Underlying Resource
        |
        +---- View

The view MUST preserve sufficient information to prevent access outside its permitted region.

A stale view MUST be rejected.

---

16. Stale Handle Protection

Where ULABI uses opaque handles, implementations SHOULD use mechanisms such as:

- generation counters;
- unique resource identifiers;
- capability tokens;
- runtime validation;
- protected handle tables.

Conceptually:

Handle {
    resource_id
    generation
}

If a resource is released and the identifier is reused:

old generation = 7
new generation = 8

a handle referencing generation 7 MUST NOT gain access to generation 8.

---

17. Use-After-Free

After successful release:

resource -> invalid

Any subsequent use MUST NOT be treated as valid.

Where detection is possible, the implementation SHOULD return a defined lifetime or resource error.

Where detection is impossible at the language level, the implementation MUST rely on an applicable boundary mechanism such as:

- capability revocation;
- handle validation;
- memory protection;
- isolation;
- reference tracking;
- sandbox termination.

---

18. Use-After-Transfer

After successful ownership transfer:

A owns R
      |
   transfer
      |
      v
B owns R

A MUST NOT continue exercising ownership rights.

An implementation MUST prevent or detect use of the transferred resource through stale ownership state.

---

19. Use-After-Expiration

A resource whose lifetime has expired MUST NOT remain accessible merely because a reference still exists.

This applies to:

- call-scoped buffers;
- borrowed values;
- temporary views;
- asynchronous resources;
- session resources;
- device mappings;
- process resources.

---

20. Use-After-Revocation

If access is revoked:

Valid
  |
revoke
  |
  v
Revoked

the revoked capability MUST no longer authorize access.

Revocation MUST take precedence over previously granted ordinary retention.

---

21. Double Release

A resource MUST NOT be successfully released twice under the same ownership contract.

The second release MUST either:

- be rejected;
- return an already-released error; or
- be explicitly idempotent under the resource contract.

It MUST NOT corrupt another resource.

---

22. Double Transfer

Ownership transfer MUST be a state transition.

A resource already transferred MUST NOT be transferred again using stale ownership authority.

A failed transfer MUST preserve the original valid ownership state unless the contract explicitly defines an atomic alternative.

---

23. Ownership Safety

This specification relies on the ownership semantics defined by:

"docs/memory/ownership.md"

Ownership itself is not redefined here.

However, any ULABI implementation MUST enforce the following safety invariant:

«At every point in time, ownership authority MUST correspond to a valid resource state.»

An implementation MUST NOT expose two independent exclusive owners simultaneously.

---

24. Lifetime Safety

This specification relies on:

"docs/memory/lifetimes.md"

A memory reference MUST remain valid only for the lifetime granted by its contract.

The existence of:

- a pointer;
- a wrapper;
- a variable;
- a handle;
- a reference count;
- a language object;

does not independently override the ULABI lifetime contract.

---

25. Allocation Safety

This specification relies on:

"docs/memory/allocation.md"

Allocation operations MUST satisfy:

- checked size arithmetic;
- alignment requirements;
- permission requirements;
- domain requirements;
- quota requirements;
- failure atomicity;
- compatible release semantics.

A failed allocation MUST NOT create a partially valid resource that can accidentally be used.

---

26. Alignment Safety

Every typed memory access MUST satisfy the alignment requirements of its ULABI representation.

An implementation MUST NOT silently treat an incorrectly aligned address as correctly aligned.

Where unaligned access is supported, its semantics MUST be explicitly defined.

---

27. Type Safety

A memory region MUST NOT be interpreted as a different ULABI type unless the contract explicitly permits that interpretation.

For example:

Bytes
  !=
String

and:

Integer representation
  !=
Arbitrary object representation

Type reinterpretation MUST be explicitly authorized.

---

28. Type Confusion

A component MUST NOT use a resource as type "T" when its authoritative contract identifies it as an incompatible type "U".

The implementation MUST prevent or detect type confusion.

Type identity SHOULD include:

- stable type identifier;
- version;
- representation contract;
- compatibility information where applicable.

---

29. Representation Safety

A native language representation MUST NOT automatically become a portable ULABI representation.

For example:

native pointer
native object header
native vtable
native allocator metadata
native garbage-collector metadata

MUST NOT be exposed as portable ULABI data unless the relevant representation is explicitly standardized.

---

30. Padding Safety

Uninitialized padding MUST NOT become semantic ULABI data.

Padding MUST NOT be transmitted, hashed, signed or compared as meaningful data unless explicitly defined by the representation contract.

This prevents:

- information disclosure;
- nondeterministic representations;
- accidental data leakage;
- incompatible binary comparisons.

---

31. Initialization Safety

A resource MUST NOT be treated as initialized semantic data before initialization has completed.

Allocation states MUST distinguish where relevant:

Uninitialized
ZeroInitialized
Initialized
PartiallyInitialized
Invalid

Partially initialized values MUST NOT be exposed as fully initialized values.

---

32. Uninitialized Memory

Uninitialized memory MUST NOT be:

- returned as valid semantic data;
- serialized;
- transmitted;
- hashed as canonical data;
- exposed across a trust boundary.

Where necessary, implementations MUST initialize or reject the operation.

---

33. Mutation Safety

A component may modify memory only when the contract grants write authority.

Read-only memory MUST NOT become writable through a type cast, wrapper, alias or handle transformation that bypasses authorization.

---

34. Aliasing Safety

Multiple references to the same resource MUST have explicitly defined aliasing semantics.

The implementation MUST distinguish at least conceptually between:

Exclusive access
Shared read access
Shared mutable access
Borrowed access

Two aliases MUST NOT silently provide conflicting authority.

---

35. Exclusive Access

When a resource is declared exclusively accessible by one component, no other component may obtain conflicting access until the exclusive access contract ends.

This applies to:

- mutable buffers;
- exclusive ownership;
- exclusive mappings;
- mutable views;
- device resources.

---

36. Shared Read Access

Multiple consumers MAY read the same immutable resource concurrently.

This is the preferred model for safe zero-copy sharing where practical.

             Immutable Resource
               /           \
              /             \
          Reader A        Reader B

---

37. Shared Mutable Access

Shared mutable memory MUST define:

- synchronization;
- visibility;
- ordering;
- atomicity;
- ownership;
- lifetime;
- mutation authority.

ULABI MUST NOT assume that shared memory is automatically thread-safe.

---

38. Data-Race Safety

Where a ULABI profile requires data-race freedom, an implementation MUST prevent or detect conflicting unsynchronized accesses.

Possible enforcement mechanisms include:

- locks;
- atomics;
- ownership;
- borrowing;
- transactional memory;
- runtime race detection;
- static analysis;
- hardware mechanisms.

The enforcement mechanism is implementation-defined.

---

39. Atomic Memory

If atomic operations are exposed, the contract MUST define:

- supported widths;
- alignment;
- atomicity;
- memory ordering;
- supported operations;
- failure semantics;
- platform limitations.

An implementation MUST NOT claim stronger atomic guarantees than it provides.

---

40. Memory Ordering

Where shared memory operations are observable between components, the applicable contract MUST define ordering semantics.

Possible ordering classes include:

- relaxed;
- acquire;
- release;
- acquire-release;
- sequentially consistent;
- implementation-defined profile semantics.

The contract MUST NOT leave synchronization semantics ambiguous.

---

41. Zero-Copy Safety

ULABI permits zero-copy interoperability only when the following are compatible:

- type;
- layout;
- alignment;
- ownership;
- lifetime;
- permissions;
- locality;
- synchronization;
- memory domain;
- representation.

Zero-copy MUST NOT be selected merely for performance if safety cannot be established.

---

42. Safe Zero-Copy Decision

Conceptually:

Zero-Copy Request
       |
       v
Type compatible?
       |
       v
Layout compatible?
       |
       v
Alignment valid?
       |
       v
Lifetime valid?
       |
       v
Ownership valid?
       |
       v
Permissions valid?
       |
       v
Synchronization valid?
       |
       v
Memory domain compatible?
       |
   +---+---+
   |       |
  YES      NO
   |       |
zero-copy copy/convert

If any mandatory condition fails, the implementation MUST copy, convert, isolate or reject the operation.

---

43. Memory Domain Safety

ULABI may distinguish:

- host memory;
- device memory;
- shared memory;
- persistent memory;
- mapped memory;
- accelerator memory;
- remote memory.

A resource MUST NOT be accessed through an incompatible memory-domain mechanism.

---

44. Address-Space Safety

A pointer valid in one address space MUST NOT automatically be interpreted as valid in another.

This applies across:

- processes;
- virtual machines;
- containers;
- devices;
- machines;
- distributed systems.

Cross-boundary access MUST use an appropriate representation such as:

- handle;
- descriptor;
- shared-memory reference;
- canonical representation;
- transport-specific resource.

---

45. Process Isolation

Where ULABI communication crosses a process boundary, one process MUST NOT obtain arbitrary access to another process's memory merely because a ULABI interface exists.

The receiving process MUST receive only the resources and capabilities explicitly granted.

---

46. Sandbox Safety

A sandboxed component MUST NOT escape its permitted memory domain through:

- raw addresses;
- forged handles;
- invalid descriptors;
- unchecked offsets;
- type confusion;
- capability forgery;
- allocator confusion.

The sandbox boundary MUST remain authoritative.

---

47. Capability Integration

Memory safety and authorization are related but distinct.

A resource may be valid but inaccessible to a particular component.

Therefore:

Valid Resource
      +
Valid Capability
      =
Authorized Access

The implementation MUST verify the applicable capability before performing privileged memory operations.

Capability semantics are defined by:

"docs/security/capability-security.md"

---

48. Capability Forgery

Opaque capabilities MUST NOT be forgeable through ordinary user-controlled data.

Implementations SHOULD use:

- cryptographically protected tokens where appropriate;
- protected handle tables;
- unguessable identifiers;
- generation counters;
- process-local authority tables;
- hardware protection.

The appropriate mechanism depends on the security profile.

---

49. Memory Revocation

If a capability is revoked, the associated access MUST cease according to the revocation contract.

Revocation MUST NOT be bypassable by retaining:

- a pointer;
- a copied handle;
- a stale wrapper;
- an alias;
- an old capability representation.

---

50. Secure Memory

Security-sensitive memory MAY require properties such as:

- zero-on-release;
- non-dumpability;
- non-swappability;
- isolation;
- protected pages;
- restricted mapping;
- restricted sharing.

If such a property is mandatory and cannot be provided, the implementation MUST reject the request rather than silently downgrade it.

---

51. Secret Data

Implementations handling security-sensitive memory SHOULD minimize:

- unnecessary copying;
- unnecessary retention;
- exposure through diagnostics;
- exposure through serialization;
- exposure through crash dumps;
- exposure through logs;
- exposure through debugging interfaces.

Secret data MUST NOT be copied into a less-protected memory domain without authorization.

---

52. Memory Disclosure

An implementation MUST prevent unintended disclosure of:

- uninitialized bytes;
- freed memory contents;
- allocator metadata;
- stale object contents;
- protected resources;
- unrelated process memory.

Memory returned to a less-trusted component MUST contain only data authorized for that component.

---

53. Memory Isolation

Faults affecting one memory resource SHOULD be contained so that they cannot silently corrupt unrelated resources.

Strong isolation SHOULD be preferred for:

- untrusted components;
- plugins;
- distributed workers;
- sandboxed code;
- incompatible runtimes;
- safety-critical components.

---

54. Fault Containment

When memory corruption is detected, the implementation SHOULD isolate the smallest affected execution domain.

Conceptually:

Memory Fault
     |
     v
Identify Scope
     |
     +---- local resource
     |
     +---- component
     |
     +---- process
     |
     +---- device
     |
     +---- system

The implementation SHOULD avoid escalating a local fault into system-wide corruption.

---

55. Failure Behavior

Memory-safety failures MUST be explicit.

Possible error classes include:

- "OutOfBounds";
- "InvalidHandle";
- "StaleHandle";
- "UseAfterRelease";
- "UseAfterTransfer";
- "LifetimeExpired";
- "CapabilityDenied";
- "TypeMismatch";
- "AlignmentViolation";
- "PermissionViolation";
- "MemoryDomainViolation";
- "OwnershipViolation";
- "ConcurrentAccessViolation";
- "InitializationViolation";
- "MemoryCorruptionDetected";
- "ResourceRevoked";
- "UnsupportedRepresentation".

The final error taxonomy MUST integrate with the ULABI exception/error model rather than creating an incompatible second error system.

---

56. No Undefined Cross-Boundary Memory Behavior

ULABI implementations MUST NOT intentionally expose language-specific undefined behavior as portable ULABI semantics.

For example, an implementation MUST NOT define:

out-of-bounds access

as a valid interoperability operation merely because one source language permits an unsafe operation internally.

Unsafe internal behavior MUST remain internal or be rejected at the ULABI boundary.

---

57. Unsafe Escape Hatches

ULABI MAY provide explicitly marked unsafe operations for implementations requiring low-level access.

Such operations MUST:

- be explicitly identified;
- declare their effects;
- declare their required capabilities;
- define their validity assumptions;
- define failure behavior;
- be excluded from safe conformance claims unless separately verified.

An unsafe operation MUST NOT silently appear as a safe ULABI operation.

---

58. FFI Safety

Foreign-function interfaces MUST validate the memory assumptions required by the foreign call.

Before crossing the FFI boundary, the implementation SHOULD validate:

- type compatibility;
- size;
- alignment;
- lifetime;
- ownership;
- mutability;
- calling convention;
- memory domain;
- representation.

The FFI MUST NOT assume that two languages use compatible memory representations merely because their source-level types appear similar.

---

59. ABI Layout Safety

For direct binary access, the applicable ABI contract MUST define:

- size;
- alignment;
- field offsets;
- field ordering;
- representation;
- padding;
- endianness;
- ownership;
- lifetime.

Native language layout MUST NOT be treated as portable without an explicit ULABI layout contract.

---

60. Serialization Safety

When memory is serialized, only semantically valid data may be serialized.

The serializer MUST NOT serialize:

- raw pointers;
- invalid handles;
- stale references;
- allocator metadata;
- uninitialized padding;
- process-local addresses;

unless an explicit profile defines a safe representation.

---

61. Distributed Memory Safety

Remote systems MUST NOT treat remote memory as ordinary local memory unless a profile explicitly defines equivalent semantics.

Remote resources MUST account for:

- failure;
- disconnection;
- expiration;
- revocation;
- identity;
- authorization;
- consistency;
- serialization;
- transport semantics.

---

62. Asynchronous Memory Safety

An asynchronous operation MUST explicitly retain or transfer any memory it will use after the initiating call returns.

The following is invalid:

submit(buffer)
return
release(buffer)
worker uses buffer

unless the asynchronous contract guarantees that the worker no longer depends on the buffer.

The correct model is:

submit(buffer)
     |
     v
retain/transfer
     |
     v
worker
     |
     v
completion
     |
     v
release retention

---

63. Cancellation Safety

Cancellation MUST define the fate of memory resources retained by an asynchronous operation.

Cancellation MUST NOT create:

- use-after-release;
- double release;
- leaked ownership;
- stale handles;
- inaccessible resources.

---

64. Exception Safety

A memory operation that fails MUST leave resources in a defined state.

Unless explicitly documented otherwise:

«Failure MUST NOT leave the caller with ambiguous ownership or an invalid resource that appears valid.»

Operations SHOULD provide failure atomicity where practical.

---

65. Transactional Memory Operations

For compound memory operations, implementations SHOULD prefer:

validate
   |
perform
   |
commit

over:

partially modify
   |
discover failure
   |
ambiguous state

If rollback is impossible, the operation MUST define its partial-failure semantics.

---

66. Recovery

Memory-safety recovery MUST be bounded and policy-controlled.

ULABI MUST NOT permit arbitrary self-modification merely because a memory fault was detected.

The preferred sequence is:

Fault detected
      |
      v
Collect evidence
      |
      v
Classify fault
      |
      v
Known recovery policy?
   +--+--+
  YES    NO
   |      |
recover  isolate/escalate
   |
verify
   |
healthy?
 +--+--+
YES    NO
 |      |
done   rollback/escalate

Recovery behavior belongs to the reliability architecture and MUST integrate with:

"docs/reliability/recovery.md"

and:

"docs/reliability/self-healing.md"

---

67. Self-Healing Safety Boundary

Self-healing mechanisms MUST NOT weaken memory-safety guarantees.

A recovery mechanism MUST NOT:

- disable memory protection merely to restore service;
- bypass capability checks;
- ignore lifetime violations;
- silently revalidate stale references;
- convert corrupted data into trusted data;
- modify safety policy without authorization.

Recovery MUST preserve the underlying ULABI contract.

---

68. Diagnostics

Memory-safety failures SHOULD expose sufficient diagnostic information for debugging without leaking protected data.

Diagnostics SHOULD identify:

- operation;
- resource identity;
- error category;
- component;
- contract;
- relevant location;
- expected bounds;
- attempted bounds;
- lifetime state;
- capability state.

Sensitive memory contents SHOULD NOT be included by default.

---

69. Deterministic Failure

Equivalent invalid operations under equivalent contract conditions SHOULD produce equivalent semantic failure categories.

The exact diagnostic text MAY differ.

The semantic error MUST remain stable enough for portable error handling.

---

70. Concurrency and Ownership

Concurrency MUST NOT implicitly alter ownership.

An implementation MUST NOT create multiple exclusive owners merely because multiple threads, tasks or processes access a resource.

Concurrency permissions MUST be explicit.

---

71. Memory Barriers and Visibility

Where multiple execution contexts share mutable memory, the applicable concurrency profile MUST define when writes become observable.

Memory safety MUST NOT depend on undocumented processor-specific behavior.

---

72. Device and Accelerator Memory

Device, GPU, NPU, FPGA or other accelerator memory MUST have explicit:

- domain;
- ownership;
- mapping;
- synchronization;
- lifetime;
- access permissions.

A host pointer MUST NOT automatically be interpreted as a valid device pointer.

---

73. DMA Safety

Where DMA-capable memory is exposed, the contract MUST define:

- ownership;
- permitted device;
- addressability;
- lifetime;
- synchronization;
- revocation;
- completion semantics.

DMA access MUST NOT continue after the memory resource becomes invalid.

---

74. Persistent Memory

Persistent-memory resources MUST distinguish:

memory validity

from:

persistence validity

A resource being persisted does not automatically make its in-memory references portable across process restarts.

Raw pointers MUST NOT be persisted as portable ULABI references.

---

75. Garbage-Collected Implementations

Garbage-collected implementations MAY implement ULABI safely.

They MUST still provide explicit semantics for:

- borrowed references;
- exported buffers;
- pinned memory where required;
- external resources;
- lifetime extension;
- ownership transfer;
- asynchronous retention.

Garbage collection MUST NOT be assumed to preserve a ULABI lifetime indefinitely.

---

76. Reference-Counted Implementations

Reference counting MAY be used internally.

Reference counting alone MUST NOT be treated as proof of:

- exclusive ownership;
- absence of cycles;
- absence of data races;
- capability validity;
- correct type;
- correct bounds.

---

77. Ownership-Based Implementations

Ownership-oriented languages MAY map their native ownership systems directly to ULABI semantics.

They MUST nevertheless expose sufficient information to other implementations to establish the boundary contract.

ULABI MUST NOT require consumers to understand the source language's ownership syntax.

---

78. Manual-Memory Implementations

Manual-memory implementations MAY use explicit allocation and release.

They MUST still satisfy the ULABI safety requirements.

The existence of an explicit "free", "delete" or equivalent operation MUST NOT excuse:

- use-after-free;
- double release;
- stale ownership;
- invalid lifetime;
- capability violations.

---

79. Managed References

Managed references MAY be represented through runtime-specific objects.

The runtime MUST prevent implementation-specific reference metadata from being confused with portable ULABI semantic data.

---

80. Memory Safety Levels

ULABI should define conformance levels.

Level 0 — Boundary Declaration

The implementation declares its memory contract but does not claim complete enforcement.

Level 1 — Validation

The implementation validates required boundary conditions.

Level 2 — Enforcement

The implementation actively prevents or detects applicable memory violations.

Level 3 — Isolation

Memory faults are contained within defined execution domains.

Level 4 — Verified Safety

Critical safety properties are supported by formal verification, certified analysis or equivalent evidence.

The exact certification criteria are defined by the ULABI standards and certification documents.

---

81. Safe Profile

A ULABI Safe Memory Profile SHOULD require:

- bounds validation;
- lifetime validation;
- ownership validation;
- type validation;
- capability validation;
- alignment validation;
- checked size arithmetic;
- stale-handle protection;
- defined failure behavior.

---

82. High-Assurance Profile

A high-assurance implementation SHOULD additionally support:

- formal invariants;
- model checking where applicable;
- static analysis;
- deterministic failure;
- fault injection;
- fuzz testing;
- memory-corruption detection;
- isolation;
- auditable evidence;
- reproducible conformance tests.

---

83. Conformance Requirements

A conforming implementation MUST pass applicable memory-safety tests for every memory profile it claims to support.

At minimum, the conformance suite SHOULD test:

1. valid access;
2. boundary access;
3. out-of-bounds access;
4. zero-length access;
5. overflowed size;
6. underflowed offset;
7. invalid alignment;
8. use-after-release;
9. use-after-transfer;
10. expired lifetime;
11. revoked resource;
12. double release;
13. double transfer;
14. stale handle;
15. type mismatch;
16. unauthorized mutation;
17. unauthorized release;
18. capability denial;
19. invalid view;
20. invalid shared access;
21. asynchronous retention;
22. asynchronous cancellation;
23. zero-copy validation;
24. cross-process isolation;
25. serialization of invalid references;
26. device-memory misuse where supported;
27. recovery behavior;
28. diagnostic behavior.

---

84. Negative Testing

The conformance suite MUST contain deliberate invalid operations.

A memory implementation MUST NOT be considered conformant solely because valid programs execute correctly.

The suite MUST demonstrate that invalid operations are:

- rejected;
- detected;
- isolated;
- or otherwise handled according to the contract.

---

85. Fuzz Testing

Memory-boundary implementations SHOULD support fuzz testing of:

- lengths;
- offsets;
- handles;
- type descriptors;
- ownership transitions;
- lifetime transitions;
- alignment;
- serialization;
- memory views;
- concurrent operations.

Fuzzing MUST include malformed and adversarial inputs.

---

86. Fault Injection

Implementations SHOULD support controlled injection of:

- allocation failure;
- invalid handles;
- expired lifetimes;
- capability revocation;
- memory-domain failure;
- device removal;
- process termination;
- asynchronous cancellation;
- corrupted descriptors.

The purpose is to verify that memory failures remain contained and deterministic.

---

87. Security Requirements

A memory-safe implementation MUST:

- enforce declared memory permissions;
- prevent capability forgery;
- prevent unauthorized resource access;
- prevent stale references from gaining authority;
- prevent accidental disclosure;
- prevent cross-domain memory confusion;
- preserve isolation boundaries;
- report security-relevant failures.

---

88. Compatibility Requirements

Memory-safety behavior MUST remain compatible across ULABI versions.

A newer implementation MUST NOT silently weaken an existing safety guarantee.

If a newer implementation requires a stronger memory capability, it MUST use feature negotiation or capability discovery.

Relevant integration documents:

- "ULABI-VERSIONING.md";
- "docs/compatibility/backwards-compatibility.md";
- "docs/compatibility/forwards-compatibility.md";
- "docs/compatibility/feature-negotiation.md";
- "docs/compatibility/capability-discovery.md";
- "docs/compatibility/graceful-degradation.md".

---

89. Graceful Degradation

If an implementation cannot provide a requested zero-copy or unsafe optimization safely, it SHOULD prefer:

zero-copy
   |
unsafe?
   |
   +-- YES --> copy/convert/isolate
   |
   +-- NO ---> zero-copy

Performance MUST NOT override safety.

---

90. Cross-Language Requirement

ULABI memory safety MUST be implementable independently by different languages.

For example:

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

may all implement ULABI independently.

ULABI MUST NOT require any one language's memory model to become the universal model.

Zamani and Sankofa remain independent implementations/targets and MUST NOT be merged into the ULABI specification.

---

91. Cross-Runtime Requirement

A ULABI implementation MUST be able to interoperate with runtimes using different memory-management strategies.

For example:

GC Runtime
    |
ULABI
    |
Ownership Runtime

must remain possible.

The boundary contract, not the internal runtime mechanism, determines interoperability.

---

92. Safety Invariants

The following invariants are normative.

Invariant 1 — Validity

A component MUST NOT access a resource that is not valid.

Invariant 2 — Bounds

A component MUST NOT access outside the authorized bounds.

Invariant 3 — Lifetime

A component MUST NOT access a resource after its authorized lifetime.

Invariant 4 — Ownership

A component MUST NOT exercise ownership rights it does not possess.

Invariant 5 — Capability

A component MUST NOT perform operations beyond its granted capabilities.

Invariant 6 — Type

A resource MUST NOT be interpreted as an incompatible type.

Invariant 7 — Alignment

Typed access MUST satisfy the applicable alignment contract.

Invariant 8 — Initialization

Uninitialized data MUST NOT be exposed as initialized semantic data.

Invariant 9 — Isolation

A memory fault MUST NOT silently corrupt unrelated trust domains.

Invariant 10 — Failure

A rejected memory operation MUST NOT silently become a successful unsafe operation.

---

93. Safety State Machine

A resource SHOULD be modeled conceptually as:

                 +-----------+
                 |  Created  |
                 +-----------+
                       |
                       v
                 +-----------+
                 |   Valid   |
                 +-----------+
                  /    |    \
                 /     |     \
                v      v      v
          Transfer   Revoke  Expire
             |         |       |
             v         v       v
           Valid     Invalid  Invalid
             |
             v
          Release
             |
             v
          Invalid

Invalid states MUST NOT transition back into valid states without a newly authorized resource creation or explicitly defined recovery transition.

A stale reference MUST NOT itself restore validity.

---

94. Recovery State Machine

If a memory fault is recoverable:

Fault
  |
  v
Detect
  |
  v
Classify
  |
  v
Isolate
  |
  v
Known Policy?
 / \
YES NO
 |   |
 v   v
Recover  Escalate
 |
 v
Verify
 |
 +----+
 |    |
OK   FAIL
 |    |
Done Rollback/Escalate

Recovery MUST NOT bypass safety invariants.

---

95. Reference Implementation Requirements

A future ULABI reference implementation SHOULD provide:

- safe handles;
- bounds checking;
- lifetime tracking;
- ownership tracking;
- capability validation;
- checked arithmetic;
- type validation;
- deterministic errors;
- memory isolation;
- test instrumentation.

The reference implementation MUST be implementation-independent from any particular language.

---

96. Required Interfaces

A future implementation SHOULD expose conceptual interfaces equivalent to:

MemoryValidator
MemoryResourceValidator
BoundsValidator
LifetimeValidator
OwnershipValidator
TypeValidator
AlignmentValidator
CapabilityValidator
HandleValidator
ViewValidator
ConcurrencyValidator
MemoryDomainValidator
SafetyPolicy
SafetyErrorReporter
FaultIsolationController
RecoveryController

These are semantic interfaces, not mandatory programming-language APIs.

---

97. Integration Contract

This document integrates with:

Core ABI

"docs/abi/core-abi.md"

Defines the general ULABI contract and stable interface semantics.

ABI Memory Model

"docs/abi/memory-model.md"

Defines memory resources, ownership, borrowing, transfer, layouts, views and address-space semantics.

Allocation

"docs/memory/allocation.md"

Defines allocation, reallocation and release contracts.

Ownership

"docs/memory/ownership.md"

Defines authoritative ownership semantics.

Lifetimes

"docs/memory/lifetimes.md"

Defines validity intervals, expiration, retention, transfer and revocation.

Shared Memory

"docs/memory/shared-memory.md"

Defines shared-memory synchronization and sharing semantics.

Virtual Memory

"docs/memory/virtual-memory.md"

Defines virtual address-space and mapping semantics.

Capability Security

"docs/security/capability-security.md"

Defines authorization and capability semantics.

Security Model

"docs/security/security-model.md"

Defines system-wide security requirements.

Exception Model

"docs/abi/exception-model.md"

Defines how memory-safety errors are represented and propagated.

FFI

"docs/interoperability/foreign-function-interface.md"

Defines language-boundary integration.

Feature Negotiation

"docs/compatibility/feature-negotiation.md"

Defines negotiation of optional memory-safety capabilities.

Conformance

"docs/standards/conformance.md"

Defines implementation conformance.

Test Suite

"docs/standards/test-suite.md"

Defines executable conformance tests.

Certification

"docs/standards/certification.md"

Defines evidence and certification requirements.

Reliability

"docs/reliability/fault-detection.md"
"docs/reliability/fault-isolation.md"
"docs/reliability/recovery.md"
"docs/reliability/self-healing.md"

define detection, containment and recovery behavior.

---

98. Ownership of Definitions

To prevent duplication:

Concept| Authoritative document
General ULABI architecture| "ULABI-DESIGN.md"
Normative ULABI rules| "ULABI-SPEC.md"
ABI memory semantics| "docs/abi/memory-model.md"
Allocation| "docs/memory/allocation.md"
Memory safety| This document
Ownership| "docs/memory/ownership.md"
Lifetimes| "docs/memory/lifetimes.md"
Shared memory| "docs/memory/shared-memory.md"
Virtual memory| "docs/memory/virtual-memory.md"
Security capabilities| "docs/security/capability-security.md"
FFI| "docs/interoperability/foreign-function-interface.md"
Error propagation| "docs/abi/exception-model.md"
Compatibility| "docs/compatibility/*"
Reliability/recovery| "docs/reliability/*"
Conformance| "docs/standards/conformance.md"
Executable tests| "docs/standards/test-suite.md"

No document should redefine another document's authoritative semantics.

---

99. Implementation Independence

A ULABI implementation MUST be able to implement these safety semantics without adopting a specific:

- programming language;
- compiler;
- runtime;
- operating system;
- CPU;
- allocator;
- garbage collector;
- ownership system;
- vendor technology.

The specification defines the contract.

The implementation defines the mechanism.

---

100. Final Principle

«ULABI memory safety means that independently implemented systems may exchange memory without requiring them to share the same internal memory-management model, while invalid, unauthorized, expired, out-of-bounds and incompatible memory operations remain prevented, detected, contained and explicitly reported.»

Safety MUST remain stronger than optimization.

Zero-copy MUST remain subordinate to safety.

Performance MUST NOT override correctness.

Recovery MUST NOT override safety.

Language-specific memory models MUST remain independent.

ULABI defines the universal boundary contract.What this file now owns

The important architectural decision is that memory-safety.md does not become a second memory model.

It owns the safety invariants and enforcement requirements:

bounds;

lifetime validity;

stale references;

use-after-free;

use-after-transfer;

type confusion;

alignment;

initialization;

aliasing;

capability enforcement;

isolation;

zero-copy safety;

concurrency safety;

async safety;

FFI safety;

fault containment;

safety recovery;

conformance requirements.


That separation is important because the existing docs/abi/memory-model.md already defines the memory abstraction and allocation.md already defines allocation semantics.


---

Files/modules that should exist

The repository currently has the major documentation tree, but the actual implementation layer is not yet present in the tree shown by the repository inspection.

I would not start implementation code immediately. First complete the independent contracts in dependency order.

Phase 1 — independent foundational specifications

These should be completed before the safety implementation:

docs/memory/ownership.md
docs/memory/lifetimes.md
docs/memory/allocation.md
docs/memory/memory-safety.md       ← this document
docs/memory/virtual-memory.md
docs/memory/shared-memory.md

allocation.md and lifetimes.md already contain substantial work, so they should be treated as existing foundations rather than rewritten unnecessarily.

Phase 2 — security contracts

docs/security/security-model.md
docs/security/capability-security.md
docs/security/sandboxing.md
docs/security/authentication.md
docs/security/authorization.md
docs/security/zero-trust.md
docs/security/secure-loading.md
docs/security/supply-chain-security.md

This is particularly important because the current capability-security.md is still a placeholder.

Phase 3 — interoperability contracts

docs/interoperability/language-interoperability.md
docs/interoperability/foreign-function-interface.md
docs/interoperability/object-model.md
docs/interoperability/name-mangling.md
docs/interoperability/symbol-resolution.md
docs/interoperability/cross-language-data.md

Phase 4 — ABI contracts

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md
docs/abi/stack-model.md
docs/abi/register-model.md
docs/abi/exception-model.md
docs/abi/return-values.md

These already exist and should be considered dependencies rather than duplicate implementation targets.


---

Required implementation modules

Once the specifications are stable, I recommend creating an implementation-independent ULABI core along these lines:

src/
└── ulabi/
    │
    ├── mod.*
    │
    ├── core/
    │   ├── interface.*
    │   ├── identifier.*
    │   ├── version.*
    │   ├── contract.*
    │   ├── capability.*
    │   └── error.*
    │
    ├── types/
    │   ├── primitive.*
    │   ├── composite.*
    │   ├── descriptor.*
    │   ├── layout.*
    │   ├── compatibility.*
    │   └── validation.*
    │
    ├── memory/
    │   ├── resource.*
    │   ├── allocation.*
    │   ├── ownership.*
    │   ├── lifetime.*
    │   ├── safety.*
    │   ├── bounds.*
    │   ├── alignment.*
    │   ├── handle.*
    │   ├── view.*
    │   ├── permissions.*
    │   ├── domain.*
    │   ├── aliasing.*
    │   ├── initialization.*
    │   └── transfer.*
    │
    ├── concurrency/
    │   ├── access.*
    │   ├── atomic.*
    │   ├── ordering.*
    │   ├── synchronization.*
    │   └── race-detection.*
    │
    ├── security/
    │   ├── capability.*
    │   ├── authorization.*
    │   ├── isolation.*
    │   ├── sandbox.*
    │   ├── revocation.*
    │   └── secure-memory.*
    │
    ├── ffi/
    │   ├── boundary.*
    │   ├── marshal.*
    │   ├── unmarshal.*
    │   ├── layout-check.*
    │   └── safety-check.*
    │
    ├── transport/
    │   ├── local.*
    │   ├── process.*
    │   ├── shared-memory.*
    │   └── remote.*
    │
    ├── runtime/
    │   ├── resource-manager.*
    │   ├── lifetime-manager.*
    │   ├── capability-manager.*
    │   └── fault-manager.*
    │
    ├── diagnostics/
    │   ├── error-report.*
    │   ├── trace.*
    │   └── audit.*
    │
    └── recovery/
        ├── fault-detection.*
        ├── isolation.*
        ├── recovery.*
        ├── rollback.*
        └── verification.*

The * deliberately means implementation language agnostic. ULABI should not decide that these modules must be Rust, C++, Go, Zamani, Sankofa, or anything else.


---

Required test modules

The implementation should eventually have an equally explicit safety test architecture:

tests/
├── memory/
│   ├── valid-access.*
│   ├── bounds.*
│   ├── overflow.*
│   ├── underflow.*
│   ├── alignment.*
│   ├── initialization.*
│   ├── type-safety.*
│   ├── use-after-release.*
│   ├── use-after-transfer.*
│   ├── lifetime.*
│   ├── stale-handles.*
│   ├── double-release.*
│   ├── aliasing.*
│   ├── zero-copy.*
│   └── views.*
│
├── ownership/
│   ├── transfer.*
│   ├── borrowing.*
│   └── exclusive-access.*
│
├── concurrency/
│   ├── races.*
│   ├── atomics.*
│   └── ordering.*
│
├── security/
│   ├── capability.*
│   ├── revocation.*
│   ├── isolation.*
│   └── disclosure.*
│
├── ffi/
│   ├── layout.*
│   ├── type-boundary.*
│   └── lifetime-boundary.*
│
├── async/
│   ├── retention.*
│   ├── cancellation.*
│   └── completion.*
│
├── recovery/
│   ├── fault-injection.*
│   ├── isolation.*
│   ├── rollback.*
│   └── verification.*
│
└── conformance/
    ├── core.*
    ├── memory.*
    ├── security.*
    ├── compatibility.*
    └── profiles.*


---

The integration rule I recommend locking in

Every future ULABI document should contain these sections:

1. Purpose
2. Scope
3. Non-goals
4. Terminology
5. Normative language
6. Fundamental principles
7. Requirements
8. Data structures
9. Interfaces
10. Invariants
11. Security requirements
12. Failure modes
13. Recovery behaviour
14. Compatibility
15. Conformance
16. Reference implementation requirements
17. Integration contract
18. Authoritative ownership of definitions
19. Test requirements
20. Implementation independence

That gives you the property you specifically requested:

> Once a file is completed, another document should not later require changing its fundamental design merely because another component was implemented.



The current repository already has the right broad architecture for this approach: ABI, architecture, compatibility, distributed, hardware, interoperability, memory, runtime, security, standards and tooling are separated into distinct areas.

