I checked the actual Benwellonedge28/ULABI repository before writing this. The repository currently has ULABI-DESIGN.md at about 70 KB, and its architecture explicitly establishes language/runtime/platform neutrality, a minimal stable Core, profiles, explicit semantics, compatibility, security, and independent implementations. It also explicitly keeps Zamani and Sankofa outside ULABI itself.

I also checked the current ABI tree. calling-convention.md, core-abi.md, data-types.md, exception-model.md, memory-model.md, register-model.md, and return-values.md already exist; stack-model.md exists but is currently only a placeholder.

The existing calling-convention design already establishes that ULABI defines semantic argument positions, while physical register/stack allocation belongs to target-specific lowering.

So stack-model.md should not duplicate the calling convention or register model. It should define the standardized semantic and physical-stack contract boundary that those documents can depend on.

Implementation order

To satisfy your requirement that a file can be completed without later reopening it merely because another component was added, I recommend this dependency order:

Phase 1 — independent normative ABI foundations

1. docs/abi/core-abi.md


2. docs/abi/data-types.md


3. docs/abi/memory-model.md


4. docs/abi/register-model.md


5. docs/abi/stack-model.md ← this document


6. docs/abi/calling-convention.md


7. docs/abi/return-values.md


8. docs/abi/exception-model.md



Phase 2 — interoperability

9. docs/interoperability/language-interoperability.md


10. docs/interoperability/foreign-function-interface.md


11. docs/interoperability/object-model.md


12. docs/interoperability/name-mangling.md


13. docs/interoperability/symbol-resolution.md


14. docs/interoperability/cross-language-data.md



Phase 3 — runtime/memory

15. docs/runtime/runtime-interface.md


16. docs/runtime/process-model.md


17. docs/runtime/threading.md


18. docs/runtime/async-model.md


19. docs/runtime/concurrency.md


20. docs/runtime/resource-management.md


21. docs/memory/memory-safety.md


22. docs/memory/ownership.md


23. docs/memory/lifetimes.md


24. docs/memory/allocation.md


25. docs/memory/virtual-memory.md


26. docs/memory/shared-memory.md



Phase 4 — compatibility, security and distributed operation

Then the compatibility, security, distributed, hardware, tooling, observability, reliability and standards documents.

This means stack-model.md is deliberately written now with all future integration points already specified.


---

docs/abi/stack-model.md

Copy everything below into that file.

# ULABI Stack Model Specification

**Document:** `docs/abi/stack-model.md`  
**Project:** ULABI  
**Full Name:** Universal Language Application Binary Interface  
**Repository:** `Benwellonedge28/ULABI`  
**Specification Family:** ABI  
**Component:** Stack Model  
**Status:** Architecture / Design Specification  
**Specification Version:** 0.1.0-draft  
**Normative Language:** MUST, MUST NOT, SHOULD, SHOULD NOT, MAY  
**License:** Apache-2.0

---

# 1. Purpose

This document defines the ULABI Stack Model.

The purpose of the Stack Model is to define the rules governing stack-like execution state at a ULABI boundary while preserving complete independence from:

- programming languages;
- compilers;
- runtimes;
- operating systems;
- CPU architectures;
- hardware vendors;
- native calling conventions;
- garbage collectors;
- ownership systems;
- register sets;
- virtual-machine implementations.

ULABI does not require every implementation to have a physical stack.

Instead, ULABI defines the semantics that a stack-capable implementation MUST preserve when a stack is used.

The Stack Model therefore provides a contract between:

```text
ULABI Semantic Call Model
          |
          v
     Stack Model
          |
          v
Target-specific lowering
          |
          v
Native ABI / Runtime / VM / IPC

The Stack Model MUST NOT turn a platform-specific stack layout into a universal ULABI requirement.


---

2. Relationship to the ULABI Architecture

ULABI follows:

> Minimal Core + Standard Profiles + Extensible Ecosystem.



The Stack Model is an ABI-level specification.

It interacts with:

ULABI Core
                         |
          +--------------+--------------+
          |              |              |
      Data Types   Calling Convention  Errors
          |              |              |
          +--------------+--------------+
                         |
                    Stack Model
                         |
          +--------------+--------------+
          |              |              |
      Register       Memory          Runtime
       Model          Model         Interface

The Stack Model defines stack semantics without requiring a universal physical stack implementation.


---

3. Scope

This specification defines:

1. stack frame semantics;


2. stack ownership;


3. stack boundaries;


4. stack growth semantics;


5. stack alignment;


6. argument spill areas;


7. local-variable areas;


8. temporary storage;


9. return-state storage;


10. saved execution state;


11. frame identity;


12. frame lifetime;


13. nested calls;


14. recursion;


15. reentrancy;


16. tail calls;


17. stack unwinding;


18. stack exhaustion;


19. guard regions;


20. stack isolation;


21. stack metadata;


22. debugging metadata;


23. security metadata;


24. asynchronous interruption;


25. cancellation;


26. exception integration;


27. coroutine integration;


28. segmented stacks;


29. dynamically growing stacks;


30. fixed-size stacks;


31. stackless implementations;


32. virtualized stacks;


33. cross-thread stack rules;


34. cross-process stack boundaries;


35. deterministic stack behavior;


36. conformance requirements.



This specification does NOT mandate one physical stack layout.


---

4. Non-Goals

This document does not define:

a universal CPU register set;

a universal instruction set;

a universal native ABI;

a universal operating-system stack;

a universal memory allocator;

a universal exception mechanism;

a universal threading implementation;

a universal coroutine implementation;

a universal debugger;

a universal virtual-machine architecture.


Those concerns are handled by their respective ULABI specifications or implementation profiles.


---

5. Fundamental Principle

The fundamental rule is:

> ULABI standardizes observable stack semantics, not one physical stack representation.



An implementation MAY represent execution state using:

a conventional downward-growing stack;

an upward-growing stack;

segmented stacks;

split stacks;

dynamically resized stacks;

fixed-size stacks;

heap-allocated frames;

continuation objects;

coroutine frames;

register windows;

virtual-machine frames;

stackless state machines;

another implementation-defined mechanism.


If the implementation exposes a ULABI-compatible call, it MUST preserve the semantic requirements defined by this specification.


---

6. Stack Abstraction

A ULABI stack is an ordered collection of execution frames.

Conceptually:

Stack
 |
 +-- Frame N
 |
 +-- Frame N-1
 |
 +-- Frame N-2
 |
 +-- ...
 |
 +-- Root Frame

The physical direction of growth is implementation-defined.

Therefore:

"higher address"

MUST NOT be treated as equivalent to:

"deeper stack"

or:

"caller frame"

without an architecture-specific definition.

ULABI defines frame relationships semantically.


---

7. Stack Frame

A stack frame represents one active invocation context.

A conceptual ULABI frame contains:

Frame {
    frame_id
    caller_frame
    callee_identity
    arguments
    local_state
    temporary_state
    return_state
    ownership_state
    capability_state
    execution_state
    metadata
}

The physical representation MAY differ.

A frame MUST have a logical identity while it is active if the implementation exposes frame identity through a ULABI profile.


---

8. Frame Identity

A frame identity MUST NOT be derived solely from a raw memory address.

A frame identifier MAY contain:

frame_id
invocation_id
function_id
execution_context_id
generation

The identifier MUST remain unambiguous within the scope in which the applicable profile exposes it.

Implementations MUST account for address reuse.

A frame identifier MUST NOT incorrectly identify a newly created frame as an old frame merely because the same physical memory was reused.


---

9. Frame Lifetime

A frame begins when a function invocation becomes active.

A frame ends when the invocation has:

returned successfully;

returned with an error;

been cancelled;

terminated exceptionally;

been unwound;

otherwise reached its defined terminal state.


After frame destruction:

borrowed references into the frame MUST NOT remain valid;

frame-local capabilities MUST NOT remain valid unless explicitly transferred;

frame-local temporaries MUST NOT be observable;

frame-local metadata MUST NOT be treated as active execution state.



---

10. Caller and Callee Relationship

Every nested invocation has a logical relationship:

Caller Frame
      |
      v
Callee Frame

The caller MAY remain active while the callee executes.

The callee MUST NOT modify caller-owned frame state unless the ULABI contract explicitly permits it.

This rule is independent of whether the implementation physically stores both frames on one stack.


---

11. Arguments

Arguments MAY be represented in:

registers;

stack memory;

descriptors;

indirect references;

handles;

implementation-defined storage.


The Stack Model is responsible only for arguments that are lowered into stack-associated storage.

The semantic argument order is defined by the Calling Convention specification.

The physical placement MUST NOT change the semantic ordering.


---

12. Stack Argument Area

An implementation MAY provide a stack argument area.

Conceptually:

+-----------------------+
| Argument N            |
+-----------------------+
| Argument N-1          |
+-----------------------+
| ...                   |
+-----------------------+
| Argument 1            |
+-----------------------+

The exact layout is target-specific.

If a ULABI profile exposes stack argument layout, that layout MUST define:

alignment;

size;

byte order;

ownership;

lifetime;

padding;

descriptor representation;

validity rules.


Undocumented padding MUST NOT contain semantically meaningful data.


---

13. Local Variables

A frame MAY contain local variables.

Local variables are owned by their frame unless explicitly transferred.

A local variable MUST NOT be accessed after the lifetime of its frame unless the value has been safely transferred to another valid storage location.

Implementations using garbage collection MAY relocate locals.

Implementations using ownership systems MAY move locals.

ULABI observes the semantic lifetime, not the physical address.


---

14. Temporary Values

A frame MAY contain temporary values used during execution.

Temporary values:

MUST have bounded lifetime;

MUST NOT become externally visible unless explicitly exported;

MUST NOT be used after invalidation;

MUST be destroyed or released according to their type and ownership semantics.


Compiler-generated temporaries are implementation details unless exposed through a debugging or introspection profile.


---

15. Return State

A frame MAY contain information required to resume the caller.

This may include:

return location;

continuation;

result storage;

caller frame identifier;

execution context;

security context.


ULABI does not mandate a hardware return-address representation.

A return state MUST remain protected from unauthorized modification.


---

16. Return Address

A native implementation MAY store a return address in:

a register;

stack memory;

a protected control-flow structure;

a shadow stack;

a continuation object;

another mechanism.


If a physical return address exists, it MUST be protected according to the applicable security profile.

ULABI MUST NOT assume that a return address is directly writable by application code.


---

17. Stack Alignment

Where a physical stack is used, the implementation MUST define an alignment requirement.

For an exposed stack ABI:

stack_alignment = N bytes

The value MUST be explicitly defined for the target ABI/profile.

An implementation MUST NOT silently violate its declared alignment requirement.

Alignment requirements apply to all exposed stack objects for which the profile specifies alignment.


---

18. Alignment and Data Types

Stack alignment MUST be compatible with the representation requirements of the values stored on the stack.

Examples include:

scalar values;

aggregates;

SIMD values;

descriptors;

pointers;

handles;

capability tokens.


A type-specific alignment requirement MUST NOT be silently reduced merely because the value happens to be placed on a stack.


---

19. Padding

Padding MAY exist for alignment.

Padding bytes:

MUST NOT be interpreted as semantic fields;

MUST NOT be used to infer type identity;

MUST NOT be relied upon for compatibility;

SHOULD be initialized deterministically when exposed across a security-sensitive boundary.


Canonical serialization MUST NOT depend on unspecified physical stack padding.


---

20. Stack Growth

A physical implementation MAY use:

downward growth;

upward growth;

segmented growth;

dynamically relocated growth;

non-contiguous growth.


ULABI does not mandate one direction.

If an implementation dynamically relocates a stack, all references into the affected stack region MUST remain valid according to the memory model or MUST be updated safely before becoming observable.


---

21. Fixed-Size Stacks

An implementation MAY use a fixed-size stack.

The implementation MUST define:

maximum stack capacity;

reserved guard region;

exhaustion detection;

failure behavior.


Stack exhaustion MUST NOT silently corrupt unrelated memory.

The preferred failure is an explicit bounded resource failure.

Example:

StackLimitExceeded


---

22. Dynamically Growing Stacks

A dynamically growing stack MAY expand when additional capacity is required.

Growth MUST be:

bounded;

detectable;

policy-controlled;

safe against integer overflow;

safe against address-space exhaustion.


The implementation MUST define behavior when growth fails.

Possible failure:

StackGrowthFailed

The implementation MUST NOT continue execution using an invalid frame.


---

23. Segmented and Split Stacks

Implementations MAY divide execution state into multiple stack segments.

Conceptually:

Segment A
    |
    +-- Segment B
            |
            +-- Segment C

Segment boundaries MUST be invisible to ordinary ULABI semantic calls.

If stack inspection is exposed, segment boundaries MUST be represented explicitly.


---

24. Stackless Implementations

A ULABI implementation MAY have no physical stack.

It MAY represent invocation state using:

heap objects;

continuations;

state machines;

coroutine frames;

runtime-managed activation records.


Such an implementation remains conformant if it provides the required ULABI semantics.

This is critical for portability to:

WebAssembly;

embedded systems;

asynchronous runtimes;

coroutine systems;

future execution architectures.



---

25. Register/Stack Boundary

The Register Model defines register semantics.

The Calling Convention defines logical argument ordering.

The Stack Model defines stack-associated execution storage.

Therefore:

ULABI Semantic Argument
          |
          +------> Register
          |
          +------> Stack
          |
          +------> Memory Descriptor
          |
          +------> Indirect Representation

The physical choice belongs to the target ABI.

A ULABI implementation MUST NOT expose target-specific register assumptions as universal stack requirements.


---

26. Spill Storage

A compiler or runtime MAY spill register values into stack storage.

Spill storage is normally implementation-private.

It MUST NOT become part of the public ABI unless explicitly specified by a debugging, tracing, or introspection profile.

Spill locations MAY change between builds.

A conforming implementation MUST NOT require another implementation to reproduce identical spill layouts.


---

27. Call Frame Layout

A target-specific exposed frame MAY conceptually contain:

+----------------------------+
| Frame Metadata             |
+----------------------------+
| Security Metadata          |
+----------------------------+
| Return State               |
+----------------------------+
| Saved Execution State      |
+----------------------------+
| Arguments / Spill Area     |
+----------------------------+
| Local Variables            |
+----------------------------+
| Temporaries                |
+----------------------------+
| Alignment / Guard Region  |
+----------------------------+

This is a conceptual model only.

An implementation MAY reorder or eliminate sections provided that all observable semantics remain valid.


---

28. Frame Pointers

An implementation MAY use a frame pointer.

A frame pointer MAY be:

a dedicated register;

a stack reference;

a metadata reference;

a runtime-managed identifier;

absent.


Frame-pointer omission MUST NOT change ULABI semantics.


---

29. Stack Pointer

A physical stack implementation normally has a stack-position concept.

The implementation MAY represent it through:

a CPU register;

a VM value;

runtime metadata;

a segmented-stack descriptor.


ULABI does not define a universal stack-pointer register.

A raw stack pointer MUST NOT be exposed across a language boundary unless explicitly permitted by a platform profile.


---

30. Stack Ownership

A stack belongs to an execution context.

Possible execution-context owners include:

thread;

coroutine;

task;

process;

virtual machine;

runtime worker.


The ownership model MUST be explicit.

One execution context MUST NOT access another execution context's private stack without an explicitly authorized mechanism.


---

31. Thread-Local Stacks

A thread MAY own a private stack.

A thread-local stack MUST NOT be assumed to be globally accessible.

If another thread requires access to data associated with the stack, the implementation MUST use an explicitly defined synchronization and ownership mechanism.


---

32. Coroutine Stacks

Coroutines MAY use:

dedicated stacks;

segmented stacks;

heap frames;

continuation objects;

stackless state machines.


A coroutine stack MUST have an explicit lifetime.

Suspending a coroutine MUST NOT invalidate values that the coroutine contract states remain live.


---

33. Asynchronous Suspension

An asynchronous operation MAY suspend its execution context.

Suspension MUST preserve all state required for later continuation.

The implementation MAY transform:

Stack Frame

into:

Continuation / Heap Frame

provided that semantic behavior remains unchanged.


---

34. Stack Switching

Stack switching MAY occur in:

coroutines;

fibers;

green threads;

virtual machines;

schedulers;

interrupt handlers.


Stack switching MUST preserve execution-context identity and security boundaries.

A stack switch MUST NOT accidentally inherit unauthorized capabilities from another execution context.


---

35. Reentrancy

A conforming implementation MUST support nested invocation according to the function's declared reentrancy semantics.

Example:

A
|
+-- B
     |
     +-- A

If a function is declared reentrant, nested execution MUST NOT corrupt its independent frame state.

If a function is declared non-reentrant, the runtime MUST enforce or expose the restriction according to the applicable profile.


---

36. Recursion

Recursive invocation is permitted.

Example:

factorial(5)
 |
 +-- factorial(4)
      |
      +-- factorial(3)
           |
           +-- ...

Each active invocation MUST have logically independent frame state.

The implementation MUST detect resource exhaustion before stack corruption occurs.


---

37. Tail Calls

A tail call MAY reuse the current frame.

If a function:

A -> B

performs a valid tail call to B, the implementation MAY replace the current frame with the callee frame.

Tail-call optimization MUST NOT alter:

ownership semantics;

capability semantics;

error semantics;

observable tracing semantics where tracing is standardized;

security semantics;

cancellation semantics.



---

38. Stack Unwinding

Stack unwinding may occur because of:

explicit error propagation;

exception handling;

cancellation;

timeout;

process termination;

resource failure.


During unwinding, resources associated with abandoned frames MUST be released according to their ownership contracts.

The implementation MUST NOT leave borrowed references pointing to destroyed frame state.


---

39. Unwinding Order

When multiple active frames are unwound:

Frame C
Frame B
Frame A

destruction occurs from the innermost active frame toward its caller:

C
↓
B
↓
A

Resources with frame-local ownership MUST be finalized before their owning frame ceases to exist.


---

40. Exception Integration

The Stack Model does not define a universal exception mechanism.

The Exception Model defines exception semantics.

When an exception crosses frames:

Caller
  |
  v
Callee
  |
  X exception
  |
  v
unwind

the Stack Model provides the frame lifetime and unwinding semantics required by the exception mechanism.

An implementation MUST NOT expose stale frame state after unwinding.


---

41. Error Result Integration

Result<T,E> does not inherently require stack unwinding.

For ordinary explicit error returns:

Result<T,E>

the frame SHOULD return normally with an error value.

This is distinct from exceptional unwinding.

The implementation MUST preserve the semantic distinction.


---

42. Cancellation

Cancellation MAY terminate an active frame.

The implementation MUST define:

cancellation observation points;

cleanup behavior;

ownership behavior;

result state;

interaction with nested calls.


A cancelled frame MUST NOT silently report successful completion.


---

43. Timeouts

A timeout MAY cause an invocation to terminate.

Timeout behavior MUST be explicit.

If the operation continues after the caller observes a timeout, the function contract MUST define that behavior.

The implementation MUST NOT assume that:

timeout == cancellation

unless the applicable profile explicitly defines them as equivalent.


---

44. Stack Guards

A physical stack SHOULD provide a guard region where practical.

A guard region exists to detect:

stack overflow;

out-of-bounds stack access;

accidental stack corruption.


Guard failures MUST be detected before unrelated memory is corrupted whenever the underlying platform permits.


---

45. Stack Exhaustion

Stack exhaustion is a resource failure.

Possible standardized error categories include:

StackLimitExceeded
StackGrowthFailed
CallDepthExceeded
ExecutionResourceExhausted

The implementation MUST NOT silently continue execution after confirmed stack exhaustion.


---

46. Stack Corruption

Stack corruption is a critical execution failure.

Possible causes include:

out-of-bounds writes;

invalid frame references;

corrupted return state;

invalid metadata;

memory safety violations.


A secure implementation SHOULD:

1. detect corruption;


2. isolate the affected execution context;


3. prevent unsafe continuation;


4. report a diagnostic;


5. terminate or recover according to policy.



Recovery MUST NOT blindly resume from corrupted control state.


---

47. Shadow Stacks

Security-oriented implementations MAY use a shadow stack.

A shadow stack may protect:

return addresses;

continuation identities;

control-flow metadata.


The shadow stack MUST remain integrity-protected.

A mismatch between normal and protected return state SHOULD be treated as a control-flow integrity violation.


---

48. Capability Context

A frame MAY carry capability state.

Example:

Frame
 |
 +-- FileRead
 +-- Network
 +-- Compute

Capabilities MUST obey the ULABI security model.

A callee MUST NOT automatically receive capabilities merely because it is nested inside a caller.

Capability delegation MUST be explicit.


---

49. Security Context

A frame MAY carry:

identity;

authorization context;

capability context;

sandbox context;

integrity level;

audit context.


Security context propagation MUST be explicit.

A stack transition MUST NOT accidentally escalate privileges.


---

50. Stack Isolation

Where execution contexts are isolated, their stacks SHOULD be isolated as well.

Isolation may be provided through:

virtual memory;

hardware protection;

runtime checks;

capability mechanisms;

process boundaries;

memory tagging;

sandboxing.


The implementation MAY use another mechanism that provides equivalent protection.


---

51. Cross-Process Calls

A process boundary MUST NOT expose a raw local stack frame to another process.

For:

Process A
    |
    | ULABI
    v
Process B

the call MUST be represented through an explicit ABI/transport mechanism.

Arguments MUST be copied, serialized, shared through an explicitly defined shared-memory mechanism, or otherwise safely transferred.


---

52. Distributed Calls

A distributed ULABI call MUST NOT expose local stack addresses as remote references.

Remote invocation MUST use the Distributed ABI and serialization/transport profiles.

The stack is always local to the execution context unless a specialized shared-execution profile explicitly defines another mechanism.


---

53. Zero-Copy Integration

Zero-copy operation MAY be supported.

However:

> Zero-copy does not mean unrestricted sharing of stack memory.



Stack memory MUST NOT be exposed for zero-copy access unless its lifetime, ownership, synchronization, and security properties are explicitly guaranteed.

Long-lived shared data SHOULD generally use heap/shared-memory/resource abstractions instead of stack storage.


---

54. Stack References

A stack reference is valid only while the referenced object remains within its defined lifetime.

A reference MUST NOT outlive the object it references.

A stack reference MUST NOT be converted into a longer-lived reference without a valid ownership or lifetime transition.


---

55. Escaping Stack Values

A value MAY escape a frame if it is safely transferred.

Example:

Frame A
 |
 +-- local value
       |
       +-- move/copy
              |
              v
          Heap / Resource

A raw pointer into a destroyed frame MUST NOT escape.

Compiler/runtime transformations MAY move the value to another storage location before returning.


---

56. Stack Pinning

An implementation MAY pin stack-associated objects.

Pinning MUST be explicitly defined where observable.

A pinned object MUST NOT be relocated while a valid external reference depends on its physical location.

Pinning MUST NOT create an implicit lifetime extension beyond the contract.


---

57. Determinism

Where deterministic execution is required, stack behavior affecting observable results MUST be deterministic.

Examples include:

initialization;

alignment;

frame identity;

unwinding order;

resource release order;

diagnostics.


Uninitialized stack data MUST NOT become observable through a ULABI boundary.


---

58. Stack Metadata

A debugging or observability profile MAY expose standardized frame metadata.

Possible metadata:

FrameMetadata {
    frame_id
    function_id
    source_location
    module_id
    invocation_id
    caller_frame_id
    stack_size
    state
}

Metadata MUST NOT expose sensitive information unless authorized.


---

59. Debugging

Debugging tools MAY inspect logical frames.

A debugger MUST distinguish:

logical frame;

physical stack frame;

optimized-away frame;

inlined function;

continuation;

coroutine frame.


An optimized implementation MUST NOT be considered non-conformant merely because its physical frame layout differs from a source-level representation.


---

60. Tracing

Tracing MAY record:

invocation start;

invocation completion;

invocation failure;

frame identity;

function identity;

nesting relationship.


Tracing MUST NOT require a specific physical stack layout.


---

61. Profiling

Profilers MAY reconstruct call relationships using:

frame metadata;

stack unwinding;

runtime events;

sampling;

instrumentation.


A profiler MUST NOT assume that every ULABI implementation uses a conventional native stack.


---

62. Stack Introspection

Stack introspection is OPTIONAL.

If implemented, it MUST expose semantic information through a versioned interface.

It MUST NOT expose arbitrary raw stack memory by default.

Raw stack access SHOULD require an explicit privileged capability.


---

63. Stack Memory Visibility

Stack memory is private execution state by default.

It MUST NOT be assumed to be:

globally visible;

remotely addressable;

shareable;

persistent;

serializable.


Any such behavior requires an explicit profile.


---

64. ABI Stability

Changes to physical stack layout MUST NOT break ULABI compatibility when the semantic contract remains unchanged.

Therefore:

Physical Stack Layout
        !=
ULABI Interface Identity

A change in:

frame padding;

spill location;

frame pointer use;

stack growth direction;

register spill strategy


does not necessarily constitute an ABI-breaking change.


---

65. Public Stack Layouts

A public stack layout MAY be defined by a platform-specific profile.

If exposed, it MUST specify:

version;

architecture;

alignment;

field layout;

offsets;

sizes;

ownership;

lifetime;

endian requirements;

security rules.


Such a layout MUST NOT be treated as universal ULABI Core semantics.


---

66. Stack and Native ABI Translation

The target adapter translates:

ULABI Frame Semantics
        |
        v
Target ABI
        |
        v
Native Frame

Examples include:

ULABI
 |
 +-- x86-64 ABI
 |
 +-- AArch64 ABI
 |
 +-- RISC-V ABI
 |
 +-- WebAssembly
 |
 +-- VM
 |
 +-- Embedded runtime

The target adapter is responsible for physical lowering.


---

67. Stack and Register Translation

The Register Model defines the register-side contract.

The Stack Model defines stack-side semantics.

The Calling Convention determines when an argument or result may use either mechanism.

The three specifications MUST be interpreted together:

Calling Convention
                       |
             +---------+---------+
             |                   |
             v                   v
       Register Model       Stack Model
             |                   |
             +---------+---------+
                       |
                       v
                Target ABI


---

68. Stack and Memory Model Integration

The Memory Model defines:

ownership;

lifetime;

references;

sharing;

mutation;

transfer;

memory safety.


The Stack Model applies those rules to stack-associated storage.

Therefore:

> A stack is a storage mechanism, not an exemption from the ULABI memory model.



Stack objects MUST obey the same fundamental lifetime and safety requirements as other memory objects.


---

69. Stack and Calling Convention Integration

The Calling Convention defines:

logical argument order;

argument semantics;

ownership;

return semantics;

effects;

execution mode.


The Stack Model defines how those semantics may be lowered into stack-associated storage.

The Calling Convention MUST remain authoritative for semantic call behavior.

The Stack Model MUST remain authoritative for stack-specific constraints.


---

70. Stack and Return Values

Return values MAY be represented through:

registers;

stack storage;

indirect memory;

descriptors;

handles;

runtime-managed values.


The Return Values specification defines semantic return behavior.

The Stack Model only defines stack-associated storage requirements.

A function MUST remain semantically compatible if the return value moves from stack storage to registers on another target.


---

71. Stack and Exception Model

The Exception Model defines the meaning of exceptional control flow.

The Stack Model defines:

frame lifetime;

frame destruction;

unwinding;

cleanup;

protection of return state.


An implementation MUST implement both specifications consistently.


---

72. Stack and Runtime Model

The Runtime Interface MAY define:

thread stacks;

task stacks;

coroutine stacks;

scheduler interaction;

stack quotas;

stack growth policies.


Runtime policies MUST NOT silently violate ABI semantics.


---

73. Resource Limits

Implementations SHOULD support explicit stack resource limits.

Possible limits include:

maximum_stack_bytes
maximum_frame_bytes
maximum_call_depth
maximum_segments
maximum_growth

Limits SHOULD be discoverable through the applicable runtime or capability interface.


---

74. Sandboxing

Sandboxed execution MAY impose stricter stack limits.

A sandbox MUST NOT allow stack growth to escape its authorized resource boundary.

Stack exhaustion inside a sandbox MUST remain contained.


---

75. Real-Time Systems

Real-time profiles MAY prohibit:

unbounded stack growth;

unpredictable frame allocation;

unbounded unwinding;

dynamic stack relocation.


A real-time implementation MUST declare the applicable restrictions.


---

76. Embedded Systems

Embedded implementations MAY use fixed-size stacks.

They MUST define:

stack capacity;

overflow detection;

maximum call depth;

interrupt interaction;

recovery behavior.


Memory-constrained implementations MAY omit optional stack metadata.


---

77. Safety-Critical Systems

Safety-critical profiles SHOULD require:

statically bounded stack usage;

verified maximum call depth;

stack overflow detection;

protected return state;

deterministic cleanup;

evidence of conformance.


The exact assurance level belongs to the applicable safety profile.


---

78. Interrupts and Exceptional Execution

Interrupt or signal handling is platform-specific.

If an interrupt creates a new execution frame, the implementation MUST protect the interrupted frame.

An interrupt handler MUST NOT corrupt the interrupted ULABI execution state.


---

79. Stack and Signals

Where operating-system signals exist, signal handling MAY use alternate stack mechanisms.

Signal-specific stack semantics are outside ULABI Core.

A platform profile MAY standardize them.


---

80. Stack and Transactions

A runtime MAY associate transactional state with a frame.

If so, transaction lifetime MUST be explicitly defined.

A stack unwind MUST NOT silently commit transactional state unless the transaction contract explicitly permits it.


---

81. Stack and Capabilities

Capability-bearing stack state MUST obey the Capability Security specification.

A capability MUST NOT become accessible merely because its token happens to be present in a stack frame.

Capability validation remains mandatory.


---

82. Stack and Secret Data

Sensitive values MAY exist in stack storage.

Security-sensitive implementations SHOULD:

minimize secret lifetime;

prevent unintended disclosure;

clear sensitive storage where required;

prevent unauthorized stack inspection.


Clearing requirements MUST be defined by the applicable security profile.


---

83. Stack Sanitization

After frame destruction, sensitive stack memory SHOULD be sanitized where required by the security profile.

Sanitization MUST NOT be assumed to provide complete security against hardware-level attacks unless the platform explicitly guarantees it.


---

84. Stack Address Disclosure

Raw stack addresses SHOULD NOT be exposed through ULABI interfaces by default.

Address disclosure MAY weaken:

memory isolation;

control-flow protection;

sandbox security;

address-space randomization.


An implementation exposing raw addresses MUST require an explicit privileged profile.


---

85. Concurrency

Multiple execution contexts MAY have independent stacks.

A stack MUST NOT be concurrently modified by multiple execution contexts unless a specialized shared-stack profile explicitly defines synchronization.

Normal ULABI code SHOULD assume stack-private execution state.


---

86. Migration

A runtime MAY migrate an execution context between:

threads;

workers;

cores;

processors;

machines in specialized distributed systems.


If migration occurs, stack state MUST remain semantically valid.

Physical stack location MAY change.


---

87. Serialization

A normal stack frame MUST NOT be serialized automatically.

If execution-state serialization is supported, the applicable profile MUST define:

frame identity;

live variables;

references;

capabilities;

resources;

continuations;

version compatibility;

security requirements.



---

88. Checkpointing

Checkpointing MAY capture execution state.

Checkpoint data MUST NOT assume that physical stack addresses remain valid after restoration.

Restoration MUST reconstruct valid execution state.


---

89. Recovery

A stack failure MAY trigger recovery.

Recovery MUST be bounded and policy-controlled.

The implementation MUST NOT arbitrarily modify execution state merely because a stack problem was detected.

Preferred model:

Failure detected
      |
      v
Collect evidence
      |
      v
Known recovery policy?
    /       \
  YES        NO
   |          |
Recover     Escalate
   |
   v
Verify
   |
   v
Healthy?
 /     \
YES     NO
 |       |
Done   Rollback/
       Terminate


---

90. Self-Healing Integration

Self-healing MUST NOT permit unrestricted stack rewriting.

A self-healing implementation MAY:

restart an execution context;

recreate a stack;

restore from a verified checkpoint;

roll back to a known-good state;

terminate the corrupted context.


It MUST NOT silently fabricate an arbitrary new frame state and claim successful continuation.


---

91. Failure Atomicity

A stack operation SHOULD be atomic with respect to its externally observable contract where practical.

If stack frame creation fails, the function MUST NOT appear to have entered a valid frame.

If frame destruction fails to release a required resource, the runtime MUST report the failure according to its resource model.


---

92. Invalid Frame State

A frame is invalid if required execution invariants no longer hold.

Examples:

invalid return state;

invalid ownership state;

corrupted metadata;

impossible frame relationship;

invalid capability context.


Execution MUST NOT continue from a known-invalid frame unless a verified recovery mechanism explicitly reconstructs valid state.


---

93. Security Invariants

A conforming implementation MUST preserve these invariants:

1. A frame MUST NOT access memory outside its authorized region.


2. A callee MUST NOT gain unauthorized caller capabilities.


3. A destroyed frame MUST NOT remain externally accessible.


4. A return state MUST be integrity protected according to the security profile.


5. Stack exhaustion MUST NOT silently corrupt unrelated memory.


6. Raw stack addresses MUST NOT become public by default.


7. Stack references MUST obey lifetime rules.


8. Stack switching MUST preserve security context.


9. Unwinding MUST release frame-owned resources correctly.


10. Recovery MUST NOT bypass memory-safety or authorization rules.




---

94. ABI Invariants

A conforming implementation MUST preserve:

1. semantic argument ordering;


2. frame lifetime semantics;


3. return semantics;


4. ownership semantics;


5. capability semantics;


6. error semantics;


7. cancellation semantics;


8. compatibility semantics;


9. deterministic behavior where required;


10. platform independence of the semantic contract.




---

95. Compatibility

A change to physical stack implementation is backward compatible when:

function semantics remain unchanged;

type semantics remain unchanged;

ownership remains unchanged;

lifetime remains unchanged;

error behavior remains compatible;

observable guarantees remain compatible.


The following MAY change without changing the ULABI interface:

stack growth direction;

frame size;

frame pointer usage;

register spilling;

stack segmentation;

physical alignment strategy, when not exposed;

compiler optimization;

frame allocation strategy.



---

96. ABI-Breaking Stack Changes

A stack change is ABI-breaking if it changes an exposed contract such as:

public frame layout;

public stack descriptor;

required alignment;

exposed calling convention;

public frame metadata;

externally visible ownership semantics;

externally visible lifetime;

security guarantees.


Such changes MUST receive a new compatible version/profile according to ULABI versioning rules.


---

97. Feature Negotiation

Stack features MAY be negotiated.

Examples:

FixedStack
DynamicStack
SegmentedStack
Stackless
ShadowStack
FrameIntrospection
CheckpointableStack
DeterministicStack

Unknown optional features MUST NOT be treated as supported.

A caller and callee MUST agree on any stack feature that is observable across the ABI boundary.


---

98. Capability Discovery

An implementation MAY expose supported stack capabilities.

Example:

StackCapabilities {
    dynamic_growth
    segmented
    shadow_stack
    frame_introspection
    checkpointing
    maximum_frame_size
    maximum_stack_size
}

Discovery MUST be versioned.


---

99. Graceful Degradation

If an optional stack feature is unavailable, the implementation MAY use a safe alternative.

Example:

Requested:
    FrameIntrospection

Unavailable:
    use opaque invocation metadata

The implementation MUST NOT silently provide weaker security or lifetime guarantees while claiming full feature support.


---

100. Conformance Levels

The Stack Model SHOULD support at least:

Level S0 — Semantic Stack Compatibility

The implementation preserves ULABI frame and lifetime semantics.

Level S1 — Native Stack Compatibility

The implementation exposes a physical stack profile.

Level S2 — Protected Stack

The implementation provides stack integrity protections.

Level S3 — Introspectable Stack

The implementation provides standardized frame metadata.

Level S4 — Verified Stack

The implementation provides evidence of bounded and verified stack behavior for applicable safety profiles.

An implementation MUST declare the level it claims.


---

101. Mandatory Conformance Tests

A conforming test suite MUST test:

1. single function invocation;


2. nested invocation;


3. recursion;


4. deep call chains;


5. stack exhaustion;


6. frame lifetime;


7. borrowed stack references;


8. escaping values;


9. ownership transfer;


10. return-state integrity;


11. argument spill;


12. alignment;


13. padding;


14. tail calls;


15. exception unwinding;


16. cancellation;


17. coroutine suspension;


18. stack switching;


19. concurrent execution contexts;


20. stack isolation;


21. invalid frame detection;


22. capability isolation;


23. security boundary enforcement;


24. deterministic behavior where required.




---

102. Negative Tests

The conformance suite MUST include invalid cases:

access after frame destruction;

stack overflow;

invalid frame identifier;

invalid return state;

unauthorized stack access;

capability leakage;

invalid stack alignment;

invalid stack reference;

corrupted frame metadata;

invalid unwind state;

stack growth beyond declared limits.


A conforming implementation MUST reject or safely contain invalid behavior.


---

103. Fuzz Testing

Stack-related parsers, descriptors and metadata SHOULD be fuzz tested.

Fuzzing SHOULD include:

malformed frame descriptors;

invalid sizes;

integer overflow;

invalid alignment;

recursive metadata;

invalid frame references;

malformed checkpoint state;

corrupted unwind information.



---

104. Formal Verification

Critical stack components SHOULD be amenable to formal verification.

Priority areas include:

frame lifetime;

bounds checking;

stack overflow detection;

ownership transitions;

unwinding;

return-state integrity;

capability isolation.



---

105. Reference Implementation Requirements

The reference implementation MUST demonstrate:

at least one conventional stack;

nested calls;

recursion;

explicit stack limits;

stack overflow detection;

frame metadata;

safe unwinding;

ownership-aware frame destruction.


The reference implementation MUST NOT be treated as the specification.

The specification remains authoritative.


---

106. Reference Implementation Independence

A second independent implementation SHOULD be capable of implementing the Stack Model without copying the reference implementation.

This is important because ULABI is intended to support independently developed implementations.


---

107. Required Schemas

The Stack Model SHOULD eventually define machine-readable schemas under:

schemas/

Required schema candidates:

schemas/stack-frame.schema.json
schemas/stack-profile.schema.json
schemas/stack-capabilities.schema.json
schemas/stack-limits.schema.json
schemas/frame-metadata.schema.json
schemas/unwind-metadata.schema.json

These schemas MUST remain subordinate to the normative specification.


---

108. Required Examples

The repository SHOULD contain:

examples/abi/stack/

with examples for:

basic-frame
nested-calls
recursive-call
stack-overflow
dynamic-stack
segmented-stack
stackless-runtime
tail-call
exception-unwind
coroutine-stack
shadow-stack

Examples are explanatory and MUST NOT override the normative specification.


---

109. Required Conformance Tests

The repository SHOULD contain:

tests/abi/stack/

with:

stack_basic
stack_nested
stack_recursive
stack_alignment
stack_limits
stack_overflow
stack_growth
stack_segmented
stackless_equivalence
stack_tail_call
stack_unwind
stack_cancellation
stack_coroutine
stack_isolation
stack_capabilities
stack_security
stack_frame_lifetime
stack_reference_safety


---

110. Required Conformance Harness

The conformance directory SHOULD contain:

conformance/abi/stack/

with:

manifest
runner
expected-results
negative-tests
resource-limit-tests
security-tests

The harness MUST report individual requirements rather than merely producing:

ULABI compatible: YES

It SHOULD produce:

ULABI Stack Model

Frame Semantics       PASS
Frame Lifetime        PASS
Alignment              PASS
Stack Limits           PASS
Overflow Detection     PASS
Unwinding              PASS
Tail Calls             PASS
Coroutines             PASS
Isolation              PASS
Security               PASS
Negative Tests         PASS


---

111. Implementation Independence

ULABI Stack Model implementations MAY be written in any suitable language.

Possible implementation languages include:

C;

C++;

Rust;

Go;

Swift;

Kotlin;

Java;

Python;

Ada;

Fortran;

Zamani;

Sankofa;

other languages.


The ULABI specification MUST NOT depend on any of these languages.


---

112. Required Integration Points

This document is designed so that later documents do not need to redefine the stack contract.

docs/abi/core-abi.md

Consumes:

frame semantics;

stack storage abstraction;

ABI compatibility rules.


docs/abi/calling-convention.md

Consumes:

argument stack placement;

frame creation;

frame lifetime;

tail calls;

stack limits.


docs/abi/register-model.md

Consumes:

register/stack boundary;

spill semantics;

saved execution state.


docs/abi/memory-model.md

Consumes:

stack ownership;

object lifetime;

references;

escaping values;

memory safety.


docs/abi/return-values.md

Consumes:

return-state semantics;

return storage;

frame termination.


docs/abi/exception-model.md

Consumes:

unwinding;

frame destruction;

cleanup.


docs/runtime/runtime-interface.md

Consumes:

stack quotas;

stack ownership;

execution contexts;

stack growth.


docs/runtime/threading.md

Consumes:

thread-local stacks;

stack ownership;

stack switching.


docs/runtime/async-model.md

Consumes:

suspension;

continuations;

coroutine frames.


docs/runtime/concurrency.md

Consumes:

concurrent execution contexts;

stack isolation.


docs/security/security-model.md

Consumes:

stack isolation;

return-state protection;

stack integrity.


docs/security/capability-security.md

Consumes:

capability context;

capability propagation;

frame isolation.


docs/security/sandboxing.md

Consumes:

stack limits;

stack resource isolation.


docs/compatibility/feature-negotiation.md

Consumes:

stack feature negotiation.


docs/compatibility/capability-discovery.md

Consumes:

stack capability discovery.


docs/compatibility/graceful-degradation.md

Consumes:

stack feature fallback.


docs/distributed/distributed-abi.md

Consumes:

prohibition of raw stack references across process/machine boundaries.


docs/distributed/serialization.md

Consumes:

execution-state serialization restrictions.


docs/observability/diagnostics.md

Consumes:

frame metadata;

stack failure diagnostics.


docs/observability/tracing.md

Consumes:

frame and invocation relationships.


docs/observability/deterministic-debugging.md

Consumes:

logical frame identity;

stack-independent debugging.


docs/standards/conformance.md

Consumes:

Stack Model conformance levels.


docs/standards/test-suite.md

Consumes:

Stack Model mandatory and negative tests.



---

113. Integration Rule

No future specification MAY silently redefine:

stack ownership;

frame lifetime;

stack reference validity;

stack overflow semantics;

frame destruction;

stack security invariants.


If a future profile needs different behavior, it MUST define an explicit extension or profile.


---

114. Implementation Rule

An implementation MUST be able to complete the Stack Model component using this document plus the referenced normative specifications.

Adding another ULABI component MUST NOT require changing the fundamental Stack Model.

New functionality SHOULD integrate through:

profiles;

extensions;

feature negotiation;

capability discovery;

versioned schemas.



---

115. Architectural Invariant

The following invariant is permanent:

> ULABI defines the semantic stack contract; target implementations define the physical stack mechanism.



Therefore:

ULABI Stack
    !=
x86 Stack
    !=
ARM Stack
    !=
RISC-V Stack
    !=
WebAssembly Stack
    !=
VM Stack
    !=
Coroutine Stack

All may implement the same ULABI semantic contract.


---

116. Final Normative Requirements

A ULABI-conforming Stack Model implementation:

MUST:

preserve frame lifetime semantics;

preserve ownership semantics;

enforce reference lifetime;

prevent stack corruption from becoming silent;

detect or safely contain stack exhaustion;

preserve call/return semantics;

support safe unwinding;

maintain security boundaries;

preserve semantic compatibility;

avoid exposing physical stack assumptions as universal ABI rules.


MUST NOT:

require a specific CPU;

require a specific register set;

require a specific stack-growth direction;

require a specific compiler;

require a specific programming language;

expose raw stack memory by default;

permit stale frame references;

silently continue after confirmed stack corruption;

grant capabilities merely through frame nesting;

treat physical stack layout as the universal ULABI ABI.


SHOULD:

provide stack limits;

provide overflow protection;

provide guard regions;

provide frame metadata;

support safe debugging;

support deterministic behavior;

support independent conformance testing.


MAY:

implement dynamic stacks;

implement segmented stacks;

implement shadow stacks;

implement stackless execution;

implement checkpointing;

implement coroutine stacks;

implement specialized hardware protections.



---

117. Completion Criteria

docs/abi/stack-model.md is considered complete when:

all normative requirements in this document are implemented or explicitly marked unsupported;

all mandatory conformance tests pass;

all negative tests are safely handled;

stack limits are documented;

frame lifetime behavior is verified;

ownership/lifetime integration is verified;

calling-convention integration is verified;

register/stack integration is verified;

exception/unwinding integration is verified;

security integration is verified;

compatibility behavior is verified;

the implementation declares its conformance level.


No later document should require modification of this specification merely because another ULABI component has been implemented.


---

118. Architectural Declaration

ULABI is the universal interoperability contract.

It does not belong to:

Zamani;

Sankofa;

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

any compiler;

any runtime;

any operating system;

any CPU;

any hardware vendor;

any company.


The Stack Model exists solely to provide a stable, language-neutral contract for stack-related execution semantics.

---

# Required code/module architecture

The important point is that **the repository currently contains the specification layer, not a complete implementation layer**. The current tree already has the ABI documents above, while the implementation directories have not yet been established as a full code implementation. 

I would therefore lock the future implementation architecture **now**, without pretending these modules already exist.

## 1. ABI-independent core modules

These should be implemented first because they become dependencies of almost everything else:

```text
src/
├── core/
│   ├── abi_identity
│   ├── interface_identity
│   ├── function_identity
│   ├── version
│   ├── feature
│   ├── capability
│   ├── error
│   ├── effect
│   └── validation

These modules must not depend on a particular CPU or language.


---

2. Type modules

src/
├── types/
│   ├── primitive
│   ├── scalar
│   ├── integer
│   ├── float
│   ├── string
│   ├── bytes
│   ├── list
│   ├── record
│   ├── enum
│   ├── variant
│   ├── option
│   ├── result
│   ├── handle
│   ├── function
│   ├── stream
│   └── future


---

3. Memory modules

src/
├── memory/
│   ├── address
│   ├── region
│   ├── allocation
│   ├── ownership
│   ├── borrowing
│   ├── lifetime
│   ├── reference
│   ├── shared_memory
│   ├── memory_safety
│   └── memory_validation

These correspond primarily to:

docs/abi/memory-model.md
docs/memory/*


---

4. Register modules

src/
├── abi/
│   ├── register/
│   │   ├── register_class
│   │   ├── register_descriptor
│   │   ├── register_file
│   │   ├── register_allocation
│   │   ├── register_save
│   │   ├── register_restore
│   │   └── register_validation

These implement the abstraction from the Register Model rather than hardcoding x86, ARM, etc.


---

5. Stack modules

This is the code architecture specifically implied by the new stack-model.md:

src/
├── abi/
│   ├── stack/
│   │   ├── stack
│   │   ├── stack_frame
│   │   ├── frame_id
│   │   ├── frame_layout
│   │   ├── frame_metadata
│   │   ├── stack_region
│   │   ├── stack_pointer
│   │   ├── stack_alignment
│   │   ├── stack_limits
│   │   ├── stack_growth
│   │   ├── stack_segment
│   │   ├── stack_guard
│   │   ├── stack_reference
│   │   ├── stack_unwind
│   │   ├── stack_checkpoint
│   │   ├── stack_capability
│   │   ├── stack_security
│   │   └── stack_validation

Core responsibilities

Module	Responsibility

stack	Abstract stack interface
stack_frame	Frame lifecycle
frame_id	Stable logical frame identity
frame_layout	Target-independent layout description
frame_metadata	Debug/observability metadata
stack_region	Stack storage regions
stack_pointer	Abstract stack-position semantics
stack_alignment	Alignment rules
stack_limits	Resource limits
stack_growth	Dynamic growth
stack_segment	Segmented stacks
stack_guard	Overflow/corruption protection
stack_reference	Stack reference lifetime
stack_unwind	Unwinding
stack_checkpoint	Optional checkpointing
stack_capability	Capability context
stack_security	Integrity/isolation
stack_validation	Runtime conformance checks



---

6. Calling convention modules

src/
├── abi/
│   ├── calling/
│   │   ├── call_contract
│   │   ├── argument
│   │   ├── argument_class
│   │   ├── argument_passing
│   │   ├── parameter
│   │   ├── return_contract
│   │   ├── calling_context
│   │   ├── call_frame
│   │   ├── tail_call
│   │   ├── callback
│   │   ├── function_reference
│   │   └── calling_validation


---

7. Return-value modules

src/
├── abi/
│   ├── return/
│   │   ├── return_value
│   │   ├── return_storage
│   │   ├── return_descriptor
│   │   ├── result_return
│   │   ├── multi_return
│   │   └── return_validation


---

8. Exception/error modules

src/
├── abi/
│   ├── error/
│   │   ├── error_code
│   │   ├── error_value
│   │   ├── exception_boundary
│   │   ├── unwind_reason
│   │   ├── cancellation
│   │   ├── timeout
│   │   ├── panic
│   │   └── error_validation


---

9. Runtime modules

src/
├── runtime/
│   ├── execution_context
│   ├── process
│   ├── thread
│   ├── task
│   ├── coroutine
│   ├── scheduler
│   ├── async
│   ├── cancellation
│   ├── resource
│   └── runtime_validation


---

10. Security modules

src/
├── security/
│   ├── capability
│   ├── authorization
│   ├── sandbox
│   ├── isolation
│   ├── stack_protection
│   ├── shadow_stack
│   ├── control_flow_integrity
│   ├── secure_loading
│   └── security_validation


---

11. Serialization/distributed modules

src/
├── distributed/
│   ├── serialization
│   ├── deserialization
│   ├── remote_call
│   ├── transport
│   ├── service_identity
│   ├── distributed_error
│   └── distributed_validation


---

12. Platform adapters

These are deliberately isolated from the universal ULABI layer:

src/
├── platforms/
│   ├── x86_64/
│   ├── aarch64/
│   ├── riscv64/
│   ├── wasm/
│   ├── embedded/
│   └── generic/

And eventually:

src/
├── platforms/
│   └── ...
│
├── architectures/
│   ├── x86/
│   ├── arm/
│   ├── riscv/
│   ├── wasm/
│   └── future/

No platform implementation should leak back into the normative Core.


---

13. Language adapters

Eventually:

implementations/
├── c/
├── cpp/
├── rust/
├── go/
├── java/
├── python/
├── swift/
├── kotlin/
├── fortran/
├── ada/
├── zamani/
└── sankofa/

These are ULABI implementations/adapters, not parts of the ULABI specification itself.

That preserves the architectural rule in ULABI-DESIGN.md: Zamani and Sankofa can implement ULABI but cannot define ULABI.


---

14. Stack test modules

The code should eventually have:

tests/
├── abi/
│   └── stack/
│       ├── basic.rs
│       ├── nested.rs
│       ├── recursive.rs
│       ├── alignment.rs
│       ├── limits.rs
│       ├── overflow.rs
│       ├── growth.rs
│       ├── segmented.rs
│       ├── stackless.rs
│       ├── tail_call.rs
│       ├── unwind.rs
│       ├── cancellation.rs
│       ├── coroutine.rs
│       ├── isolation.rs
│       ├── capability.rs
│       ├── security.rs
│       ├── lifetime.rs
│       └── invalid_state.rs

The exact programming language for the test harness can be selected later; these are module responsibilities, not a mandate to use Rust.


---

15. Machine-readable stack schemas

Create eventually:

schemas/
├── abi/
│   ├── stack-frame.schema.json
│   ├── stack-profile.schema.json
│   ├── stack-capabilities.schema.json
│   ├── stack-limits.schema.json
│   ├── frame-metadata.schema.json
│   └── unwind-metadata.schema.json


---

16. Reference implementation

The reference implementation should eventually contain:

reference/
├── abi/
│   ├── core/
│   ├── types/
│   ├── memory/
│   ├── registers/
│   ├── stack/
│   ├── calling/
│   ├── returns/
│   └── errors/

The stack implementation specifically needs:

reference/abi/stack/
├── stack
├── frame
├── limits
├── growth
├── alignment
├── unwind
├── guard
├── metadata
└── validation


---

The key dependency rule

The architecture should ultimately be:

ULABI SPECIFICATION
                           |
                           v
                     Semantic Core
                           |
        +------------------+------------------+
        |                  |                  |
      Types             Memory            Errors
        |                  |                  |
        +------------------+------------------+
                           |
                    Calling Contract
                           |
              +------------+------------+
              |                         |
         Register Model            Stack Model
              |                         |
              +------------+------------+
                           |
                    Runtime Adapter
                           |
                  Platform Adapter
                           |
                  Native Environment

This prevents the common mistake of making the stack model depend on x86, ARM, C, Rust, or one particular compiler.

Most important consequence

docs/abi/stack-model.md is now a complete normative contract in its own right. Later documents should reference and integrate with it, not redefine it.

The repository's current stack-model.md is only a tiny placeholder, so replacing it with the specification above is appropriate.

And because the existing calling-convention.md already separates semantic ULABI arguments from physical register/stack lowering, the new Stack Model fits the architecture without creating a conflict.