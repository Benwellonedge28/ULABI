Yes. I inspected the actual ULABI repository first, rather than treating the proposed directory tree as authoritative.

The repository confirms that ULABI-DESIGN.md is the current architectural authority and is a substantial 0.2.0-draft document. It establishes the critical separation between ULABI and individual languages, the Minimal Core + Standard Profiles + Extensible Ecosystem architecture, transport independence, locality semantics, capability security, compatibility, and distributed interoperability. [ULABI-DESIGN.md](https://github.com/Benwellonedge28/ULABI/blob/main/ULABI-DESIGN.md?utm_source=chatgpt.com)

I also checked the existing hardware area. docs/hardware/cpu.md is currently only a placeholder, as are the GPU and the other hardware-specialization documents, so CPU is genuinely the correct place to establish the CPU-specific contract rather than duplicating an existing specification. [cpu.md](https://github.com/Benwellonedge28/ULABI/blob/main/docs/hardware/cpu.md?utm_source=chatgpt.com) [gpu.md](https://github.com/Benwellonedge28/ULABI/blob/main/docs/hardware/gpu.md?utm_source=chatgpt.com)

The existing calling-convention specification already establishes the important boundary: ULABI defines semantic argument/call semantics; target adapters lower those semantics into physical registers, stacks, memory, native ABIs, IPC, or distributed mechanisms. [calling-convention.md](https://github.com/Benwellonedge28/ULABI/blob/main/docs/abi/calling-convention.md?utm_source=chatgpt.com)

The existing Distributed ABI likewise explicitly treats the distributed layer as a profile above Core and delegates serialization, remote calls, service discovery, errors, and consensus boundaries to their respective specifications. [distributed-abi.md](https://github.com/Benwellonedge28/ULABI/blob/main/docs/distributed/distributed-abi.md?utm_source=chatgpt.com)

So I would not put networking, serialization, GPU semantics, or a particular CPU's native ABI into this document.

Below is the complete document for:

docs/hardware/cpu.md

You can copy it directly.


---

ULABI CPU Hardware Interface Specification

Document: docs/hardware/cpu.md
Project: ULABI
Full Name: Universal Language Application Binary Interface
Specification Family: Hardware / CPU
Component: CPU Interface
Status: Architecture / Design Specification
Specification Version: 0.1.0-draft
ULABI Architecture Version: 0.2.0-draft
Normative Authority: ULABI-DESIGN.md, ULABI-SPEC.md, ULABI-VERSIONING.md
License: Apache-2.0


---

1. Purpose

This document defines how ULABI interfaces with CPU architectures.

The purpose is not to create one universal CPU instruction set.

The purpose is to define the contract between:

ULABI Semantic ABI
        |
        v
CPU Architecture Adapter
        |
        v
Native CPU ABI
        |
        v
CPU / Hardware

ULABI therefore remains independent of:

x86;

x86-64;

ARM;

AArch64;

RISC-V;

PowerPC;

MIPS;

SPARC;

LoongArch;

S390;

future CPU architectures;

any particular CPU vendor;

any particular operating system;

any particular compiler.


The CPU layer translates the universal ULABI contract into the physical capabilities and constraints of a target CPU.


---

2. Fundamental Principle

The fundamental rule is:

> ULABI defines semantic execution contracts; CPU adapters define how those contracts are physically realized on a target processor.



Therefore:

ULABI Function
     |
     +---- x86-64 lowering
     |
     +---- AArch64 lowering
     |
     +---- RISC-V lowering
     |
     +---- PowerPC lowering
     |
     +---- future CPU lowering

All implementations MUST preserve the same ULABI semantics.

Different CPU architectures MAY use:

different registers;

different stack layouts;

different instruction sets;

different calling conventions;

different alignment requirements;

different pointer widths;

different atomic instructions;

different cache architectures;

different memory-ordering mechanisms;

different exception/trap mechanisms.


These differences MUST remain below the ULABI semantic boundary unless explicitly exposed by a CPU-specific profile.


---

3. Scope

This document defines the ULABI CPU interface for:

1. CPU architecture identification;


2. execution modes;


3. word size;


4. address width;


5. pointer representation;


6. integer representation;


7. floating-point capabilities;


8. register abstraction;


9. argument lowering;


10. return-value lowering;


11. stack interaction;


12. alignment;


13. endianness;


14. instruction-set capabilities;


15. atomic operations;


16. memory ordering;


17. memory barriers;


18. synchronization;


19. exception and trap boundaries;


20. privilege levels;


21. virtualization;


22. hardware feature discovery;


23. CPU capability negotiation;


24. deterministic execution requirements;


25. resource limits;


26. real-time constraints;


27. architecture-specific extensions;


28. heterogeneous CPU systems;


29. debugging metadata;


30. conformance testing.



This document does not define:

a universal instruction set;

a universal machine-code format;

a universal operating system;

a universal CPU;

a specific compiler backend;

a specific native ABI;

a specific assembler syntax;

a specific processor vendor.



---

4. Relationship to Other ULABI Specifications

The CPU specification is subordinate to the ULABI semantic ABI.

The relationship is:

ULABI-DESIGN.md
       |
ULABI-SPEC.md
       |
ULABI Core ABI
       |
Calling Convention
       |
CPU Interface
       |
CPU Architecture Adapter
       |
Native CPU ABI
       |
Hardware

The CPU specification depends conceptually on:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/memory-model.md

docs/runtime/runtime-interface.md

docs/tooling/compiler-interface.md
docs/tooling/linker-interface.md
docs/tooling/loader-interface.md

docs/security/security-model.md
docs/security/capability-security.md

docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

docs/standards/conformance.md
docs/standards/test-suite.md

It MUST NOT duplicate the complete rules of those specifications.

Instead, this document defines how CPU-specific implementations consume them.


---

5. CPU Architecture Identity

Every CPU target MUST have an explicit architecture identity.

Conceptually:

CPUArchitectureIdentity {
    architecture
    variant
    revision
    word_size
    address_width
    endianness
    abi_profile
    feature_set
}

Example:

architecture: AArch64
variant: generic
word_size: 64
address_width: 64
endianness: little

Another implementation might report:

architecture: RISC-V
variant: RV64
word_size: 64
address_width: 64
endianness: little

The architecture identity MUST NOT be inferred solely from:

executable filename;

operating-system name;

compiler name;

source-language name.



---

6. CPU Identity vs CPU Capability

ULABI MUST distinguish:

CPU Architecture

from:

CPU Capabilities

For example:

Architecture:
    AArch64

Capabilities:
    Atomic64
    SIMD
    FP64
    Crypto
    Virtualization

Two CPUs MAY share the same architecture while exposing different optional capabilities.

Therefore:

> Architecture compatibility does not automatically imply feature compatibility.




---

7. Word Size

The CPU adapter MUST expose the CPU word size.

Examples:

32-bit
64-bit

The word size MUST NOT redefine ULABI semantic integer types.

For example:

ULABI UInt32
ULABI UInt64

remain exactly those semantic types regardless of whether the CPU uses 32-bit or 64-bit general-purpose registers.

A CPU adapter MUST NOT silently convert:

UInt64

into:

UInt32

because the target CPU is 32-bit.

The implementation MUST either:

lower it safely;

emulate it;

reject the contract;

or use a compatible fallback.



---

8. Address Width

The CPU adapter MUST distinguish:

word size

from:

address width

A CPU MAY have:

64-bit registers

while supporting an address space smaller than the full register width.

ULABI pointer and address semantics MUST therefore be defined by the memory model rather than inferred from CPU register size.


---

9. Pointer Representation

A CPU adapter MAY represent pointers using:

native pointers;

capability pointers;

handles;

descriptors;

tagged pointers;

indirect references.


ULABI MUST NOT require a pointer to be represented as a raw machine integer.

A ULABI pointer crossing a language boundary MUST preserve the semantic memory contract.

A pointer MUST NOT become an unrestricted capability merely because it is represented by a machine address.


---

10. Null and Invalid References

Where applicable, the CPU adapter MUST distinguish:

valid reference
null reference
invalid reference
unmapped reference
revoked reference

ULABI semantic types such as:

Option<T>
Handle
Reference
Capability

MUST NOT be conflated with raw CPU addresses.


---

11. Endianness

The CPU adapter MUST explicitly report byte order.

Supported conceptual values include:

LittleEndian
BigEndian
MixedEndian
ArchitectureDefined

ULABI canonical serialization MUST remain independent of CPU endianness.

Therefore:

CPU representation
        !=
ULABI canonical encoding

A little-endian CPU and big-endian CPU MUST be capable of exchanging compatible ULABI values without changing their semantic meaning.


---

12. Alignment

The CPU adapter MUST expose relevant alignment requirements.

Conceptually:

AlignmentRequirements {
    scalar
    pointer
    aggregate
    vector
    atomic
}

ULABI structures MUST NOT depend on undocumented native alignment.

When a ULABI value is lowered to a CPU representation, the adapter MUST satisfy the target CPU's alignment requirements.

If direct alignment cannot be preserved safely, the implementation MUST use an appropriate alternative such as:

copying;

padding;

indirect representation;

aligned allocation.



---

13. Register Model

ULABI does not define one universal register set.

Instead, the CPU adapter exposes semantic register classes.

Initial classes:

GeneralPurpose
FloatingPoint
Vector
Predicate
Special
Control
Capability

A CPU MAY support only a subset.

Example:

GeneralPurpose:
    integer values
    pointers
    handles

FloatingPoint:
    floating-point values

Vector:
    SIMD values

The physical register names remain architecture-specific.


---

14. Register Allocation

Register allocation is implementation-defined.

The ULABI calling convention specifies logical arguments.

The CPU backend determines whether an argument is placed into:

a register;

multiple registers;

stack memory;

a temporary;

an indirect descriptor.


For example:

ULABI:

calculate(
    a: Int64,
    b: Int64
)

may become:

CPU:
    register 1 = a
    register 2 = b

or:

CPU:
    stack[0] = a
    stack[8] = b

Both are valid if the semantic contract is preserved.


---

15. Argument Lowering

Every CPU adapter MUST implement a deterministic argument-lowering procedure.

Conceptually:

ULABI Type
    |
    v
Type Classification
    |
    v
CPU Representation
    |
    v
Register / Stack / Memory

The lowering rules MUST account for:

type size;

alignment;

register availability;

aggregate representation;

ownership;

borrowing;

indirect arguments;

return conventions;

target ABI restrictions.



---

16. Scalar Arguments

Scalar ULABI values SHOULD be lowered directly when the CPU provides a compatible representation.

Examples:

UInt32
Int32
UInt64
Int64
Float32
Float64
Bool

However, semantic width MUST remain authoritative.

A CPU with 64-bit registers MUST NOT reinterpret UInt32 as UInt64 where that changes the contract.


---

17. Aggregate Arguments

Aggregates include:

Record
Tuple
Variant
List descriptor
Map descriptor
Option
Result

The CPU adapter MAY lower aggregates as:

register values
multiple registers
stack values
memory references
indirect descriptors

The choice MUST be deterministic for a given CPU ABI profile.


---

18. Large Arguments

Large values SHOULD normally be passed indirectly where direct register passing is inefficient or impossible.

Conceptually:

Caller
  |
  +---- pointer/descriptor ----> argument storage

The memory contract MUST define:

ownership;

lifetime;

mutability;

alignment;

accessibility;

synchronization.


A raw pointer MUST NOT be used without the required memory semantics.


---

19. Return Values

The CPU adapter MUST define deterministic return lowering.

Return values MAY use:

registers;

multiple registers;

caller-provided storage;

descriptors;

handles.


The semantic return type remains defined by the ULABI function contract.

Example:

Result<Int64, Error>

MUST NOT become architecture-specific error semantics merely because one CPU uses status registers.


---

20. Error and Status Registers

Some CPUs expose condition codes or status registers.

These MAY be used internally.

They MUST NOT replace the ULABI error model unless a specific profile explicitly defines such behavior.

For example:

CPU condition code

is not automatically equivalent to:

Result<T,E>

The adapter performs the translation.


---

21. Stack Interaction

ULABI does not require a stack.

A CPU implementation MAY use:

hardware stack;

software stack;

register-only calling;

shadow stack;

segmented stack;

split stack;

runtime-managed stack.


The physical stack model remains target-specific.

Where the CPU requires a stack, the adapter MUST define:

stack alignment;

frame rules;

spill rules;

call depth constraints;

overflow behavior;

security requirements.



---

22. Stack Safety

A conforming implementation MUST detect or safely handle stack exhaustion according to its runtime profile.

Possible outcomes include:

StackOverflow
ResourceLimit
CallDepthExceeded
ExecutionFailure

Stack exhaustion MUST NOT silently corrupt another ULABI object or execution context.


---

23. Instruction Set

ULABI MUST NOT prescribe one instruction set.

A CPU adapter MUST identify the instruction-set level required by an implementation.

Conceptually:

InstructionSetProfile {
    base
    extensions
    revision
}

Example:

Base:
    RV64I

Extensions:
    M
    A
    F
    D
    C

The exact architecture-specific naming belongs to the target implementation.


---

24. Optional CPU Features

Optional features MUST be explicitly discoverable.

Possible capabilities include:

SIMD
Vector
FloatingPoint
FP16
FP32
FP64
Atomic
Crypto
Compression
Virtualization
MemoryTagging
PointerAuthentication
TransactionalMemory
HardwareRandomness
PerformanceCounters

An implementation MUST NOT execute an optional instruction unless the required capability has been established.


---

25. Capability Discovery

CPU feature discovery integrates with:

docs/compatibility/capability-discovery.md

Conceptually:

ULABI Interface
      |
      v
Required CPU Features
      |
      v
Capability Discovery
      |
   +--+--+
   |     |
 Supported  Unsupported
   |          |
 Execute    Negotiate/
            Fallback/
            Reject

Unsupported CPU capabilities MUST NOT be silently assumed.


---

26. Feature Negotiation

CPU feature negotiation integrates with:

docs/compatibility/feature-negotiation.md

A function MAY declare:

requires:
    SIMD
    FP64

If the CPU does not support those capabilities, the implementation MUST:

1. use an approved compatible implementation;


2. use a software fallback;


3. negotiate a compatible profile;


4. or reject the operation.



It MUST NOT execute unsupported instructions.


---

27. Graceful Degradation

CPU-specific degradation MUST integrate with:

docs/compatibility/graceful-degradation.md

Example:

Vector implementation
        |
        v
CPU supports Vector?
    /       \
  YES       NO
   |         |
Vector     Scalar
path       fallback

The fallback MUST preserve semantic correctness.

Performance may decrease.

Semantics MUST NOT change unless the contract explicitly permits different semantics.


---

28. Atomic Operations

The CPU adapter MUST expose supported atomic capabilities.

Conceptually:

AtomicCapabilities {
    width
    load
    store
    compare_exchange
    exchange
    fetch_add
    fetch_sub
    bitwise
}

Supported widths and operations MAY vary by CPU.

ULABI synchronization semantics remain authoritative.


---

29. Memory Ordering

CPU adapters MUST map ULABI memory-ordering requirements to the target CPU.

Conceptual ordering levels include:

Relaxed
Acquire
Release
AcquireRelease
SequentiallyConsistent

The CPU adapter MAY use stronger native ordering than required.

It MUST NOT use weaker ordering that violates the ULABI contract.


---

30. Memory Barriers

Where required, the CPU implementation MUST provide appropriate barriers.

Conceptually:

ULABI Ordering Requirement
          |
          v
CPU Memory Barrier
          |
          v
Observable Memory Semantics

The exact CPU instruction is architecture-specific.


---

31. Atomic Failure

Atomic operations MAY fail because of:

unsupported width;

unavailable hardware capability;

alignment;

contention;

resource exhaustion.


Failure behavior MUST be explicit.

An implementation MUST NOT silently substitute a non-atomic operation where atomicity is required.


---

32. Floating-Point Capabilities

CPU floating-point support MUST be explicitly identified.

The adapter MUST account for:

precision;

rounding;

NaN;

infinity;

signed zero;

denormal/subnormal handling;

exception modes where relevant.


ULABI floating-point semantics remain defined by the type/ABI specifications.

The CPU adapter performs the physical realization.


---

33. SIMD and Vector Processing

SIMD/vector execution MAY be used as an optimization or explicit capability.

A vector-capable CPU MAY expose:

Vector<T>

only when the applicable ULABI profile supports it.

A CPU adapter MUST distinguish:

semantic vector type

from:

physical SIMD register

The latter is an implementation detail.


---

34. Determinism

Where a ULABI contract requires deterministic execution, CPU-specific behavior MUST be controlled.

Potential sources of nondeterminism include:

floating-point environment;

timing;

unordered atomics;

hardware randomness;

concurrent execution;

speculative execution effects where observable;

implementation-specific instructions.


A deterministic ULABI profile MUST define which CPU features are permitted.


---

35. Hardware Randomness

Hardware random-number instructions MAY be exposed through an appropriate randomness/capability profile.

They MUST NOT be treated as deterministic entropy.

A CPU adapter MUST distinguish:

deterministic computation

from:

hardware-generated randomness


---

36. Privilege Levels

CPUs MAY expose privilege levels such as:

user
supervisor
hypervisor
machine
secure

ULABI does not define a universal privilege hierarchy.

The adapter MUST expose only the privilege-sensitive operations permitted by the execution environment.

A ULABI component MUST NOT obtain additional CPU privilege merely by declaring a hardware capability.


---

37. Privileged Instructions

Privileged instructions MUST be treated as capability-controlled operations.

Examples include operations affecting:

memory management;

interrupt configuration;

CPU control;

device configuration;

virtualization;

performance counters.


Access MUST be governed by the applicable security and capability profiles.


---

38. Virtualization

ULABI implementations MAY run inside:

virtual machines;

hypervisors;

containers;

trusted execution environments.


The visible CPU architecture MAY be virtualized.

Therefore:

ULABI CPU Identity

MUST describe the execution CPU interface visible to the implementation, not necessarily the physical host CPU.


---

39. CPU Feature Virtualization

A hypervisor MAY expose a virtual CPU with a restricted feature set.

The ULABI implementation MUST use the advertised feature set.

It MUST NOT assume that physical host capabilities are available to the guest.


---

40. Heterogeneous CPU Systems

Systems MAY contain multiple CPU types.

Examples:

big + little cores
CPU + accelerator
heterogeneous clusters
multi-architecture distributed systems

ULABI MUST distinguish:

interface compatibility

from:

execution capability compatibility

An operation MAY migrate between CPUs only when its semantic and capability requirements remain satisfied.


---

41. CPU Affinity

CPU affinity MAY be exposed through an execution or real-time profile.

ULABI Core MUST NOT require a specific CPU affinity mechanism.

If affinity is observable or required for correctness, it MUST be declared.


---

42. Real-Time CPU Execution

A real-time profile MAY specify:

maximum latency;

bounded execution;

interrupt constraints;

scheduling guarantees;

CPU affinity;

priority;

deadline behavior.


The CPU specification provides the hardware interface.

The runtime specification defines scheduling semantics.


---

43. Performance Counters

Performance counters MAY be exposed through an observability profile.

They MUST NOT be assumed to be identical across CPU architectures.

A portable ULABI performance metric MUST define semantic meaning independently from the physical counter.


---

44. Debugging

CPU implementations SHOULD expose sufficient metadata for:

stack unwinding;

register inspection;

breakpoint mapping;

source mapping;

exception diagnosis;

call tracing.


The debugger interface is defined by:

docs/tooling/debugger-interface.md

This document only defines the CPU-side requirements.


---

45. Exception and Trap Boundary

CPU traps MAY result from:

invalid instructions;

memory faults;

alignment faults;

arithmetic faults;

privilege violations;

breakpoint instructions;

hardware exceptions.


These are CPU events.

They MUST be translated into ULABI/runtime error semantics through the applicable exception model.

A raw CPU trap MUST NOT automatically become a language-level exception.


---

46. Fault Isolation

A CPU fault MUST NOT automatically compromise unrelated ULABI components.

Where the runtime provides isolation, faults SHOULD be contained within the smallest appropriate execution domain.

Possible boundaries include:

thread
process
sandbox
virtual machine
hardware partition


---

47. Security

CPU implementations MUST integrate with the ULABI security model.

Relevant CPU security capabilities MAY include:

memory protection;

execute permissions;

privilege separation;

hardware isolation;

pointer authentication;

memory tagging;

secure execution;

virtualization isolation;

cryptographic acceleration.


Hardware security features MUST NOT automatically grant application-level authority.


---

48. Speculative Execution

CPU microarchitectural behavior MUST NOT alter the architectural ULABI semantics.

Where speculative execution creates security concerns, the implementation SHOULD use appropriate platform security mechanisms.

ULABI MUST remain concerned with observable contract semantics rather than exposing undocumented microarchitectural behavior.


---

49. Cache Behavior

CPU caches are normally implementation details.

ULABI MUST NOT require a particular:

cache size;

cache hierarchy;

cache coherence protocol;

cache line size.


However, cache coherence MAY become relevant to shared-memory or concurrency profiles.

Those semantics MUST be explicitly specified where observable.


---

50. Shared Memory

CPU shared-memory behavior MUST integrate with:

docs/memory/shared-memory.md

and the applicable concurrency specification.

A shared memory region MUST define:

ownership;

lifetime;

access rights;

synchronization;

visibility;

consistency;

invalidation behavior.



---

51. CPU-Specific ABI Profiles

A ULABI implementation MAY define architecture profiles.

Examples:

ULABI-CPU-x86-64
ULABI-CPU-AArch64
ULABI-CPU-RV64
ULABI-CPU-PPC64

Such profiles MUST specify:

architecture identity;

supported types;

register classes;

alignment;

stack rules;

argument lowering;

return lowering;

atomic capabilities;

floating-point capabilities;

vector capabilities;

feature negotiation;

exception/trap mapping.


They MUST NOT redefine ULABI semantic types.


---

52. Native ABI Adapter

The CPU layer SHOULD be implemented as an adapter between ULABI and the native ABI.

ULABI
                |
        CPU Architecture Adapter
                |
          Native ABI Adapter
                |
              CPU

The adapter MUST isolate architecture-specific assumptions from the ULABI Core.


---

53. Native ABI Independence

A CPU architecture MAY have multiple native ABIs.

For example, the same CPU architecture may support different operating systems or runtime environments.

Therefore:

CPU Architecture
       !=
Native ABI

ULABI MUST model these as separate layers.


---

54. CPU Adapter Interface

A reference CPU adapter SHOULD expose an interface conceptually equivalent to:

trait CpuTarget {
    architecture() -> ArchitectureIdentity

    capabilities() -> CpuCapabilities

    word_size() -> WordSize

    address_width() -> AddressWidth

    endianness() -> Endianness

    registers() -> RegisterModel

    classify_argument(type) -> ArgumentClass

    lower_argument(argument) -> LoweredArgument

    lower_return(type) -> ReturnConvention

    alignment(type) -> Alignment

    atomic_capabilities() -> AtomicCapabilities

    memory_ordering() -> MemoryOrderingCapabilities

    instruction_set() -> InstructionSetProfile

    validate_contract(contract) -> ValidationResult
}

The exact programming language used to implement this interface is not part of the ULABI specification.


---

55. CPU Capability Descriptor

A canonical implementation SHOULD represent CPU capabilities using a structured descriptor.

Conceptually:

CpuCapabilities {
    architecture
    variant
    revision

    word_size
    address_width
    endianness

    integer_widths
    floating_point_formats

    register_classes
    vector_widths

    atomic_widths
    memory_orderings

    instruction_extensions

    virtualization
    security_features

    hardware_randomness
}


---

56. Contract Validation

Before executing a ULABI contract, the CPU adapter SHOULD validate:

Architecture
Type Support
Alignment
Instruction Features
Register Constraints
Atomic Requirements
Floating-Point Requirements
Vector Requirements
Security Requirements
Execution Requirements

If a requirement cannot be satisfied, the implementation MUST NOT silently execute incompatible code.


---

57. CPU Capability Mismatch

A capability mismatch MUST produce an explicit result.

Conceptually:

CPU capability required
        |
        v
Capability available?
      /     \
    YES      NO
     |        |
  execute   fallback /
             negotiate /
             reject

Possible errors include:

UnsupportedArchitecture
UnsupportedInstructionSet
UnsupportedFeature
UnsupportedAtomicOperation
UnsupportedFloatingPoint
UnsupportedVectorOperation
UnsupportedExecutionMode


---

58. Compatibility

CPU compatibility MUST be evaluated at multiple levels:

Semantic Compatibility
        |
ABI Compatibility
        |
CPU Capability Compatibility
        |
Native ABI Compatibility

Two systems MAY have:

same ULABI contract
different CPU architecture

and remain compatible through an appropriate implementation.


---

59. Cross-Architecture Execution

A ULABI implementation MAY communicate between:

x86-64
AArch64
RISC-V

or other architectures.

The semantic contract remains unchanged.

Architecture differences MUST be resolved through:

compatible lowering;

serialization;

emulation;

interpretation;

recompilation;

remote execution;

another approved mechanism.



---

60. CPU Migration

If an execution context moves from one CPU architecture to another, the implementation MUST verify:

interface compatibility;

type compatibility;

execution requirements;

CPU feature requirements;

memory requirements;

security requirements.


CPU migration MUST NOT silently invalidate a ULABI contract.


---

61. Unsupported CPU Architecture

An implementation encountering an unsupported architecture MUST fail explicitly.

It MUST NOT:

guess the architecture;

assume x86 semantics;

assume little-endian behavior;

assume 64-bit pointers;

assume a particular register model.



---

62. Security Invariants

The CPU layer MUST preserve these invariants:

Invariant 1 — No implicit privilege escalation

A ULABI contract MUST NOT gain CPU privilege merely because it executes on a capable processor.

Invariant 2 — No unsafe feature assumption

Unsupported hardware capabilities MUST NOT be used.

Invariant 3 — No semantic type corruption

CPU lowering MUST preserve ULABI semantic types.

Invariant 4 — No invalid memory access

CPU-specific lowering MUST respect the ULABI memory contract.

Invariant 5 — No silent ABI mismatch

Native ABI differences MUST NOT silently alter ULABI semantics.

Invariant 6 — No architecture-specific Core dependency

The ULABI Core MUST remain architecture-neutral.


---

63. Failure Modes

CPU integration MUST explicitly account for:

UnsupportedArchitecture
UnsupportedFeature
UnsupportedInstruction
UnsupportedAtomicOperation
UnsupportedFloatingPoint
UnsupportedVectorOperation
AlignmentFailure
StackOverflow
MemoryFault
PrivilegeViolation
IllegalInstruction
ExecutionFault
ResourceExhaustion
ContractMismatch
ABIIncompatibility

Each failure MUST be translated into the appropriate ULABI/runtime error representation.


---

64. Recovery Behaviour

CPU recovery MUST remain bounded.

A CPU fault MUST NOT trigger arbitrary self-modification.

Where recovery is safe:

CPU failure
    |
    v
Capture fault
    |
    v
Classify
    |
    v
Known recovery policy?
   /      \
 YES       NO
 |          |
recover    escalate
 |
verify
 |
healthy?
 /     \
YES     NO
 |       |
continue rollback/escalate

This integrates with the Reliability and Self-Healing profiles.


---

65. Self-Healing Restrictions

The CPU layer MUST NOT independently modify:

executable code;

ABI contracts;

security policy;

capability policy;

interface identity;

memory ownership rules


merely because a CPU failure occurred.

Any recovery action MUST be explicitly authorized by the applicable runtime/reliability policy.


---

66. Deterministic CPU Lowering

For a given:

ULABI contract
+
CPU architecture
+
CPU ABI profile
+
feature set

the lowering process SHOULD be deterministic.

This enables:

reproducible builds;

ABI verification;

conformance testing;

binary compatibility analysis;

debugging;

formal verification.



---

67. Conformance Requirements

A CPU implementation conforms to this specification only if it can demonstrate:

Architecture

explicit architecture identity;

explicit word size;

explicit address width;

explicit endianness.


Calling

deterministic argument lowering;

deterministic return lowering;

valid register/stack mapping.


Types

correct semantic type preservation;

correct aggregate handling;

correct alignment.


Memory

correct pointer semantics;

correct memory ordering;

correct atomic behavior.


Features

explicit CPU feature discovery;

explicit unsupported-feature behavior.


Security

no implicit privilege escalation;

correct capability handling.


Failure

explicit architecture failures;

explicit CPU feature failures;

safe trap/error translation.


Compatibility

cross-architecture semantics preserved;

native ABI differences isolated.



---

68. CPU Conformance Tests

The ULABI test suite SHOULD contain CPU-specific tests for:

test_architecture_identity
test_word_size
test_address_width
test_endianness
test_scalar_lowering
test_aggregate_lowering
test_large_argument_lowering
test_return_lowering
test_alignment
test_register_mapping
test_stack_mapping
test_atomic_operations
test_memory_ordering
test_feature_discovery
test_feature_negotiation
test_unsupported_instruction
test_unsupported_feature
test_privilege_boundary
test_exception_mapping
test_cross_architecture_compatibility
test_deterministic_lowering

Architecture-specific test vectors SHOULD be separated from semantic ULABI tests.


---

69. Reference CPU Adapter

The reference implementation SHOULD provide a CPU-neutral interface and architecture-specific backends.

Conceptually:

reference/
└── cpu/
    ├── model
    ├── lowering
    ├── validation
    └── profiles

Architecture-specific implementations MAY eventually include:

x86_64
aarch64
riscv64
ppc64

The existence of a reference backend MUST NOT make any particular architecture normative for ULABI.


---

70. Required Implementation Boundary

The implementation architecture SHOULD be:

ULABI Contract
      |
      v
Semantic ABI
      |
      v
CPU Target Interface
      |
      +------------------+
      |                  |
 x86-64 Adapter     AArch64 Adapter
      |                  |
 Native ABI          Native ABI

This separation is REQUIRED conceptually even if an implementation combines modules internally.


---

71. Required Code Modules

The eventual implementation SHOULD contain these logical modules.

Core CPU model

cpu/
├── architecture
├── identity
├── capabilities
├── word_size
├── address_width
├── endianness
└── registers

CPU lowering

cpu/lowering/
├── arguments
├── returns
├── aggregates
├── scalars
├── references
├── handles
└── calling_frame

CPU memory interface

cpu/memory/
├── alignment
├── atomics
├── ordering
├── barriers
└── faults

CPU instruction capabilities

cpu/instructions/
├── base
├── extensions
├── features
├── validation
└── dispatch

CPU security

cpu/security/
├── privilege
├── isolation
├── memory_protection
└── hardware_capabilities

CPU compatibility

cpu/compatibility/
├── feature_detection
├── feature_negotiation
├── fallback
└── architecture_validation

Architecture adapters

cpu/targets/
├── x86_64/
├── aarch64/
├── riscv64/
└── future/

These are logical modules, not a requirement that ULABI be implemented in one particular programming language.


---

72. Required Schemas

The implementation should eventually define machine-readable schemas for:

schemas/cpu-architecture.schema.*
schemas/cpu-capabilities.schema.*
schemas/cpu-registers.schema.*
schemas/cpu-feature-set.schema.*
schemas/cpu-abi-profile.schema.*
schemas/cpu-lowering.schema.*

The actual schema format can be selected by the repository's schema policy.


---

73. Required Test Files

The implementation should eventually include:

tests/cpu/
├── architecture_identity.*
├── capabilities.*
├── scalar_lowering.*
├── aggregate_lowering.*
├── register_lowering.*
├── stack_lowering.*
├── return_lowering.*
├── alignment.*
├── atomics.*
├── memory_ordering.*
├── feature_negotiation.*
├── unsupported_features.*
├── privilege.*
├── faults.*
└── cross_architecture.*

Architecture-specific tests should be grouped beneath their target where necessary:

tests/cpu/x86_64/
tests/cpu/aarch64/
tests/cpu/riscv64/


---

74. Integration Contract

This document is intentionally designed so that later documents do not need to rewrite its CPU fundamentals.

The integration points are fixed in advance.

docs/abi/calling-convention.md

Consumes:

register classes;

argument lowering;

return lowering;

stack interaction.


It MUST NOT redefine CPU identity.

docs/abi/data-types.md

Defines semantic types.

This document defines how those types are physically lowered.

It MUST NOT redefine semantic type meaning.

docs/abi/memory-model.md

Defines ownership and memory semantics.

This document maps those semantics onto CPU memory capabilities.

docs/memory/shared-memory.md

Defines shared-memory semantics.

This document provides CPU-level ordering, atomicity, and barrier mechanisms.

docs/runtime/runtime-interface.md

Defines runtime execution.

This document supplies CPU execution capabilities and faults.

docs/security/security-model.md

Defines security semantics.

This document supplies hardware-level enforcement mechanisms.

docs/compatibility/feature-negotiation.md

Consumes CPU capability descriptors and negotiation results.

docs/compatibility/graceful-degradation.md

Consumes unsupported CPU feature information and defines fallback behavior.

docs/tooling/compiler-interface.md

Consumes CPU target information when compiling/lowering ULABI contracts.

docs/tooling/linker-interface.md

Consumes CPU architecture and native ABI identity for linking.

docs/tooling/loader-interface.md

Consumes CPU architecture, executable format, feature requirements, and runtime compatibility.

docs/standards/conformance.md

Defines how CPU implementations prove conformance.

docs/standards/test-suite.md

Provides the common conformance vectors.


---

75. Integration Rule

The following ownership boundary is permanent:

ULABI Core
    |
    | defines semantic contract
    v
Calling Convention
    |
    | defines semantic invocation
    v
CPU Specification
    |
    | defines CPU realization
    v
Native ABI
    |
    | defines platform-specific realization
    v
CPU Hardware

No lower layer may redefine the semantics of an upper layer.


---

76. What This Document Must Not Duplicate

To prevent architectural repetition, cpu.md MUST NOT become the owner of:

universal types;

generic type compatibility;

language FFI;

serialization;

remote calls;

service discovery;

distributed errors;

consensus;

authentication policy;

authorization policy;

runtime scheduling;

garbage collection;

application-level error handling;

self-healing policy;

compiler architecture.


Those belong to their respective ULABI specifications.


---

77. Architectural Invariants

The CPU specification establishes these permanent invariants:

CPU Architecture != ULABI
CPU ABI != ULABI
Native ABI != ULABI
Instruction Set != ULABI
CPU Vendor != ULABI
Operating System != ULABI

And:

ULABI Semantic Contract
        |
        v
CPU-specific realization

not:

CPU architecture
        |
        v
ULABI semantics


---

78. Final Principle

> ULABI must be able to run on a CPU without becoming defined by that CPU.



A CPU adapter is therefore an implementation boundary, not a definition of ULABI itself.

The same ULABI contract MUST remain meaningful across:

x86
ARM
RISC-V
PowerPC
MIPS
SPARC
LoongArch
future architectures

provided an implementation can satisfy the required ULABI semantics.


---

79. Status

This document establishes the architectural CPU contract.

It is intentionally written before individual CPU backends are implemented.

The next implementation work SHOULD therefore proceed from:

CPU semantic model
        ↓
CPU capability descriptor
        ↓
CPU target interface
        ↓
generic lowering engine
        ↓
architecture-specific adapters
        ↓
conformance tests

rather than beginning with one CPU architecture and accidentally making it the ULABI standard.


---

Required files and implementation order

This is the part I would lock in now so that you can work file by file without repeatedly reopening earlier files.

Phase 1 — independent foundation files

These establish concepts that other files consume.

1. docs/hardware/cpu.md

Status: this document.

Owns:

CPU identity;

CPU capabilities;

word/address width;

endianness;

register abstraction;

CPU feature model;

CPU lowering boundary;

CPU-specific failure/security invariants.


2. schemas/cpu-architecture.schema.*

Owns:

architecture identity;

variant;

revision;

word size;

address width;

endianness.


3. schemas/cpu-capabilities.schema.*

Owns:

hardware capability descriptors;

instruction extensions;

atomic capabilities;

floating-point capabilities;

vector capabilities;

security capabilities.


4. reference/cpu/model.*

Owns:

architecture model;

capability model;

register classes;

CPU identity.


These four should be completed before architecture-specific backends.


---

Phase 2 — generic CPU implementation

5. reference/cpu/target.*

The universal CPU target interface.

6. reference/cpu/lowering/arguments.*

Owns semantic-to-physical argument lowering.

7. reference/cpu/lowering/returns.*

Owns return-value lowering.

8. reference/cpu/lowering/aggregates.*

Owns records, variants, options, results and other aggregate lowering.

9. reference/cpu/lowering/scalars.*

Owns scalar lowering.

10. reference/cpu/lowering/calling_frame.*

Owns the abstract physical call-frame representation.

11. reference/cpu/validation.*

Validates whether a ULABI contract can execute on a CPU target.

These should be designed so that architecture-specific code consumes them rather than reimplementing them.


---

Phase 3 — CPU memory primitives

12. reference/cpu/memory/alignment.*

13. reference/cpu/memory/atomics.*

14. reference/cpu/memory/ordering.*

15. reference/cpu/memory/barriers.*

16. reference/cpu/memory/faults.*

These connect the CPU model to the memory/runtime specifications.


---

Phase 4 — capability and compatibility

17. reference/cpu/compatibility/feature_detection.*

18. reference/cpu/compatibility/feature_negotiation.*

19. reference/cpu/compatibility/fallback.*

20. reference/cpu/compatibility/architecture_validation.*

These integrate with:

docs/compatibility/capability-discovery.md
docs/compatibility/feature-negotiation.md
docs/compatibility/graceful-degradation.md


---

Phase 5 — architecture-independent tests

21. tests/cpu/architecture_identity.*

22. tests/cpu/capabilities.*

23. tests/cpu/scalar_lowering.*

24. tests/cpu/aggregate_lowering.*

25. tests/cpu/register_lowering.*

26. tests/cpu/stack_lowering.*

27. tests/cpu/return_lowering.*

28. tests/cpu/alignment.*

29. tests/cpu/atomics.*

30. tests/cpu/memory_ordering.*

31. tests/cpu/feature_negotiation.*

32. tests/cpu/unsupported_features.*

33. tests/cpu/privilege.*

34. tests/cpu/faults.*

35. tests/cpu/cross_architecture.*


---

Phase 6 — architecture-specific implementations

Only after the generic CPU contract is frozen should these be created:

reference/cpu/targets/x86_64/
reference/cpu/targets/aarch64/
reference/cpu/targets/riscv64/
reference/cpu/targets/ppc64/

Each target should contain, as necessary:

identity.*
registers.*
features.*
calling_convention.*
lowering.*
atomics.*
ordering.*
security.*
validation.*

This prevents x86-64 or AArch64 from contaminating the universal specification.


---

Phase 7 — toolchain integration

These are later integration modules:

compiler/
    cpu_target.*
    instruction_selection.*
    abi_lowering.*

linker/
    cpu_target.*
    relocation.*
    symbol_abi.*

loader/
    cpu_validation.*
    cpu_features.*
    executable_target.*

validator/
    cpu_contract.*
    cpu_capabilities.*

The exact top-level paths should be reconciled with the repository's existing tooling implementation before creating them; do not create duplicate compiler/linker/loader modules if equivalent modules already exist.


---

Phase 8 — conformance integration

Eventually:

conformance/cpu/
├── architecture/
├── capabilities/
├── calling/
├── types/
├── memory/
├── atomics/
├── features/
├── faults/
├── security/
└── cross_architecture/

And the final conformance report can expose:

ULABI Core          ✓
ULABI Types         ✓
ULABI Memory        ✓
ULABI Calling       ✓
ULABI CPU           ✓
ULABI Security      ✓
ULABI Async         ✓
ULABI Distributed   ✓


---

Important repository conclusion

The current repository is still in a design/specification-first state for this area: docs/hardware/cpu.md is only a placeholder, while the master design and several ABI/distributed documents already contain substantive architecture. [ULABI repository](https://github.com/Benwellonedge28/ULABI?utm_source=chatgpt.com)

Therefore, do not start writing CPU implementation modules yet. The correct order is:

cpu.md → CPU schemas → generic CPU model → generic lowering → CPU capability/compatibility → generic tests → architecture adapters → compiler/linker/loader integration → conformance.

That ordering gives you the property you asked for: when cpu.md is completed, later files consume its already-defined contracts rather than forcing you to go back and redesign the CPU specification.