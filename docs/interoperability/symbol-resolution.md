I inspected the actual Benwellonedge28/ULABI repository before drafting this. The master architecture establishes ULABI as language-, runtime-, OS-, CPU-, and vendor-neutral, with the Minimal Core + Standard Profiles + Extensible Ecosystem architecture.

I also checked the existing interoperability material. name-mangling.md already establishes the important distinction between source names, ULABI identities, and platform linkage names, while symbol-resolution.md is currently only a placeholder.   The repository also already separates calling convention, data types, memory, distributed ABI, and compatibility responsibilities, so this specification should coordinate those systems rather than duplicate them.

docs/interoperability/symbol-resolution.md

Copy the following into that file:

# ULABI Symbol Resolution Specification

Project: ULABI  
Full Name: Universal Language Application Binary Interface  
Document: `docs/interoperability/symbol-resolution.md`  
Specification Family: Interoperability  
Component: Symbol Resolution  
Specification Version: 0.1.0-draft  
ULABI Architecture Version: 0.2.0-draft  
Status: Normative Design Specification / Pre-Implementation  
License: Apache-2.0  

Normative authorities:

- `ULABI-DESIGN.md`
- `ULABI-SPEC.md`
- `ULABI-VERSIONING.md`

Related specifications:

- `docs/interoperability/language-interoperability.md`
- `docs/interoperability/foreign-function-interface.md`
- `docs/interoperability/name-mangling.md`
- `docs/interoperability/object-model.md`
- `docs/interoperability/cross-language-data.md`
- `docs/abi/core-abi.md`
- `docs/abi/calling-convention.md`
- `docs/abi/data-types.md`
- `docs/abi/exception-model.md`
- `docs/compatibility/backwards-compatibility.md`
- `docs/compatibility/forwards-compatibility.md`
- `docs/compatibility/feature-negotiation.md`
- `docs/compatibility/capability-discovery.md`
- `docs/compatibility/graceful-degradation.md`
- `docs/security/security-model.md`
- `docs/security/secure-loading.md`
- `docs/standards/conformance.md`

---

# 1. Purpose

This specification defines the ULABI symbol-resolution model.

Symbol resolution is the process by which an implementation takes a referenced ULABI entity and determines the concrete implementation capable of satisfying that entity's contract.

The process may occur:

- during compilation;
- during static linking;
- during dynamic linking;
- during loading;
- during runtime lookup;
- through a local registry;
- through an IPC mechanism;
- through a distributed service registry;
- through an accelerator/device registry.

The fundamental model is:

```text
Source Declaration
       |
       v
ULABI Interface Identity
       |
       v
Symbol Reference
       |
       v
Resolution
       |
       +-------------------+
       |                   |
       v                   v
Local Implementation   Remote/Virtual Implementation
       |                   |
       +---------+---------+
                 |
                 v
          Contract Validation
                 |
                 v
             Invocation

ULABI symbol resolution MUST resolve contracts, not merely strings.

A matching textual symbol is insufficient to establish interoperability.


---

2. Fundamental Principle

The fundamental rule is:

> Symbol resolution MUST establish that a candidate implementation satisfies the referenced ULABI contract before the candidate is used.



Therefore:

Symbol Name Match
       !=
Contract Match

A platform linker may find a symbol with the expected spelling while the implementation has:

incompatible parameter types;

incompatible return types;

incompatible ownership;

incompatible effects;

incompatible version;

incompatible ABI profile;

incompatible security requirements;

incompatible execution semantics.


Such an implementation MUST NOT be treated as a valid ULABI resolution merely because its native symbol name matches.


---

3. Separation of Identity and Resolution

ULABI defines three separate concepts:

ULABI Identity
                      |
                      v
               Symbol Reference
                      |
                      v
                Resolution
                      |
                      v
              Implementation

3.1 Identity

Identity answers:

> Which ULABI entity is being requested?



Identity is governed primarily by:

docs/interoperability/name-mangling.md

3.2 Reference

A reference represents a request to use an identified entity.

3.3 Resolution

Resolution answers:

> Which available implementation satisfies that entity's contract?



3.4 Implementation

The implementation is the concrete executable/provider representation.

It may be:

a function;

a method;

a library export;

a runtime object;

an IPC endpoint;

a service;

a device operation;

an accelerator kernel;

another ULABI provider.



---

4. Scope

This specification defines:

1. symbol references;


2. symbol identities;


3. resolution requests;


4. resolution candidates;


5. namespaces;


6. symbol kinds;


7. interface identity;


8. version matching;


9. ABI-profile matching;


10. type-contract matching;


11. capability requirements;


12. effect requirements;


13. locality requirements;


14. provider selection;


15. deterministic resolution;


16. ambiguity handling;


17. weak and optional symbols;


18. aliases;


19. redirects;


20. lazy resolution;


21. eager resolution;


22. dynamic resolution;


23. static resolution;


24. runtime registries;


25. distributed resolution boundaries;


26. security validation;


27. trust validation;


28. integrity validation;


29. failure behavior;


30. diagnostics;


31. caching;


32. invalidation;


33. compatibility;


34. conformance requirements.



This specification does NOT define:

programming-language syntax;

native linker formats;

CPU instruction sets;

object-file formats;

serialization formats;

transport protocols;

cryptographic primitives;

memory-management algorithms;

language-specific object layouts.


Those responsibilities belong to their respective ULABI specifications.


---

5. Resolution Model

A resolution request is conceptually:

ResolutionRequest {
    interface_identity
    symbol_identity
    requested_version
    required_profiles
    required_capabilities
    required_effects
    locality_requirement
    security_requirements
    execution_requirements
}

The resolver produces:

ResolutionResult {
    selected_provider
    selected_symbol
    resolved_version
    provided_profiles
    provided_capabilities
    execution_location
    validation_status
}

Or:

ResolutionFailure {
    reason
    diagnostics
    candidates_considered
    compatibility_information
}


---

6. Symbol Reference

A ULABI symbol reference MUST contain enough information to identify the required contract.

Conceptually:

SymbolReference {
    namespace
    interface
    symbol
    contract
    version
    profile_requirements
}

A symbol reference SHOULD additionally support:

capability_requirements
security_requirements
locality
execution_requirements
optionality

A resolver MUST NOT infer security-sensitive requirements from an arbitrary native symbol name.


---

7. Symbol Identity

Symbol identity is governed by:

docs/interoperability/name-mangling.md

A resolver MUST consume the canonical identity produced by that specification.

The resolver MUST NOT invent a competing identity algorithm.

The relationship is:

Canonical Semantic Identity
          |
          v
     Symbol Reference
          |
          v
        Resolver


---

8. Symbol Kinds

The resolver MUST distinguish at least:

Function
Method
Constructor
Destructor
Type
Constant
Variable
Interface
Resource
Service
Capability
Module
Namespace

For example:

function:add
constant:add
variable:add
type:add

are different symbols.

A resolver MUST NOT silently reinterpret one symbol kind as another.


---

9. Interface Resolution

An interface provides a stable contract boundary.

Example:

Interface:
    org.example.storage
    Version:
        1.x

Symbols:
    open
    read
    write
    close

The resolver MUST first establish the interface identity before resolving individual operations where the interface contract requires it.

Conceptually:

Interface
   |
   +--> Contract
   |
   +--> Version
   |
   +--> Profiles
   |
   +--> Symbols

An implementation providing only one operation MUST NOT automatically be treated as a provider of the complete interface unless the interface explicitly permits partial implementation.


---

10. Candidate Provider

A candidate provider is an implementation that claims to provide a requested ULABI interface or symbol.

A provider record SHOULD contain:

ProviderDescriptor {
    provider_identity
    interfaces
    symbols
    versions
    profiles
    capabilities
    security_metadata
    integrity_metadata
    locality
    execution_modes
}

The provider descriptor MUST be independently verifiable.


---

11. Candidate Discovery

Candidates may be discovered through:

static link metadata;

object files;

shared libraries;

runtime registries;

process-local registries;

operating-system loaders;

IPC registries;

device registries;

service registries;

explicit provider configuration;

embedded metadata;

implementation-defined discovery mechanisms.


Discovery MUST NOT itself imply trust.

The correct sequence is:

Discover
   |
   v
Inspect
   |
   v
Validate
   |
   v
Select
   |
   v
Resolve


---

12. Discovery vs Resolution

Discovery answers:

> What providers exist?



Resolution answers:

> Which provider satisfies this request?



These operations MUST remain distinct.

Example:

Provider A -> discovered
Provider B -> discovered
Provider C -> discovered

        |
        v

Compatibility evaluation

        |
        v

Provider B -> selected


---

13. Resolution Algorithm

A conforming resolver MUST conceptually perform the following sequence:

1. Validate request
2. Validate identity
3. Discover candidates
4. Filter by symbol kind
5. Filter by interface identity
6. Filter by contract identity
7. Filter by version compatibility
8. Filter by profile compatibility
9. Filter by type compatibility
10. Filter by capability requirements
11. Filter by security requirements
12. Filter by locality requirements
13. Filter by execution requirements
14. Validate provider integrity/trust
15. Rank remaining candidates
16. Detect ambiguity
17. Select provider
18. Produce binding
19. Cache if permitted

A resolver MUST NOT skip contract validation merely because a native linker has already found a matching symbol.


---

14. Determinism

Given:

the same resolution request;

the same provider set;

the same provider metadata;

the same compatibility rules;

the same security policy;


a resolver SHOULD produce the same result.

If multiple candidates are equally valid, the resolver MUST have an explicitly defined deterministic tie-breaking rule or MUST report ambiguity.

It MUST NOT make an arbitrary undocumented selection.


---

15. Candidate Filtering

Candidate providers SHOULD be filtered in the following order:

Identity
   ↓
Kind
   ↓
Contract
   ↓
Version
   ↓
Profiles
   ↓
Types
   ↓
Capabilities
   ↓
Security
   ↓
Locality
   ↓
Execution semantics
   ↓
Trust / Integrity

Implementations MAY optimize this process provided the observable result remains equivalent.


---

16. Contract Compatibility

A provider is compatible only if its exported contract satisfies the requested contract.

Compatibility MAY include:

parameter compatibility;

return compatibility;

error compatibility;

ownership compatibility;

lifetime compatibility;

effect compatibility;

execution semantics;

profile requirements;

capability requirements;

version requirements.


The resolver MUST use the authoritative specifications for each property.

It MUST NOT redefine these semantics locally.


---

17. Version Resolution

Version resolution is governed by:

ULABI-VERSIONING.md

and:

docs/compatibility/backwards-compatibility.md

docs/compatibility/forwards-compatibility.md

docs/compatibility/feature-negotiation.md


A resolver MUST distinguish:

Exact version
Compatible version
Incompatible version
Unknown version

A resolver MUST NOT silently substitute an incompatible version.

Example:

Requested:
storage.read v2

Candidate:
storage.read v1

The resolver may select v1 only if the compatibility rules explicitly permit it.


---

18. Profile Resolution

ULABI uses profiles for advanced capabilities.

Example:

Required:
Core
Streaming
Security
ZeroCopy

A provider that implements:

Core
Streaming
Security

does not satisfy the request if ZeroCopy is mandatory.

Optional profiles MAY be omitted when graceful degradation is explicitly permitted.


---

19. Capability Resolution

Capabilities are not equivalent to symbols.

A provider may implement a symbol but lack the authority required to execute it.

Conceptually:

Symbol
  |
  v
Contract
  |
  v
Capability Requirement
  |
  v
Authorization
  |
  v
Execution

Capability discovery is governed by:

docs/compatibility/capability-discovery.md

Security authorization is governed by:

docs/security/security-model.md

A resolver MUST NOT grant capabilities merely because a provider advertises them.


---

20. Security Validation

Before selecting a provider, a security-sensitive implementation SHOULD validate:

provider identity;

artifact integrity;

signature/status where applicable;

trust policy;

required capabilities;

sandbox requirements;

permitted locality;

permitted execution mode.


The resolver MUST treat untrusted provider metadata as untrusted input.

Malformed metadata MUST NOT cause unsafe execution.


---

21. Secure Loading

Symbol resolution and loading are separate stages:

Resolve
   |
   v
Validate
   |
   v
Authorize
   |
   v
Load
   |
   v
Bind

A resolver MUST NOT cause executable code to be loaded merely because an untrusted provider claims a matching symbol.

Integration with secure loading is defined by:

docs/security/secure-loading.md


---

22. Native Linker Integration

ULABI may operate above native linkers.

For example:

ULABI Identity
      |
      v
ULABI Resolver
      |
      v
Native Linker Symbol
      |
      v
Native Address

The native symbol is an implementation detail.

The resolver MUST retain the original ULABI identity independently of the native linkage name.


---

23. Name-Mangling Integration

docs/interoperability/name-mangling.md defines how ULABI identities may be represented as linkage names.

Symbol resolution MUST consume that representation.

It MUST NOT assume that a platform symbol is itself globally meaningful.

Therefore:

ULABI Identity
    !=
Native Linker Name


---

24. Foreign-Function Interface Integration

The FFI specification defines how a language invokes an already-resolved ULABI operation.

The division of responsibility is:

Symbol Resolution
    |
    | Which implementation?
    v
Resolved Symbol
    |
    | How is it invoked?
    v
FFI

The resolver MUST NOT implement language-specific calling semantics.


---

25. Language Interoperability Integration

Language interoperability defines how language implementations map their internal representations to ULABI contracts.

The resolver simply consumes the resulting ULABI identity and contract.

Language A
   |
   v
Language Adapter
   |
   v
ULABI Symbol
   |
   v
Resolver
   |
   v
Provider


---

26. Static Resolution

Static resolution occurs before execution.

Examples:

compile-time linking;

static library selection;

build-time provider selection.


Static resolution SHOULD produce diagnostics when a requested symbol cannot be resolved.

Example:

ULABI-SYMBOL-NOT-FOUND

Interface:
    org.example.math

Symbol:
    add

Contract:
    v2

No compatible provider was found.


---

27. Dynamic Resolution

Dynamic resolution occurs at load or runtime.

Examples:

dlopen-style loading
runtime registries
plugin systems
device discovery
service discovery

Dynamic resolution MUST apply the same contract-validation rules as static resolution.

Runtime resolution MUST NOT weaken security merely because the symbol was not known at compile time.


---

28. Lazy Resolution

Implementations MAY defer resolution until first use.

Example:

Program Start
      |
      v
No resolution yet
      |
      v
First call
      |
      v
Resolve
      |
      v
Validate
      |
      v
Bind

If resolution fails, the failure MUST be explicit and diagnosable.

A lazy resolver MUST NOT convert an unresolved required symbol into undefined behavior.


---

29. Eager Resolution

Implementations MAY resolve all required symbols before execution.

This is useful for:

deterministic startup;

safety-critical systems;

embedded systems;

early failure detection;

certification.


A profile MAY require eager resolution.


---

30. Optional Symbols

A symbol MAY be declared optional.

Example:

Symbol:
    accelerator.optimize

Required:
    false

If the symbol cannot be resolved, the implementation may continue only if the contract explicitly defines fallback behavior.

An optional symbol MUST NOT silently become required.


---

31. Weak Symbols

ULABI MAY support weak bindings for implementation-specific compatibility.

However, weak resolution MUST NOT override an explicitly required symbol.

A weak candidate may be selected only when:

1. no stronger valid candidate exists;


2. the contract permits fallback;


3. security policy permits the fallback;


4. the resulting behavior is defined.




---

32. Aliases

Aliases may map multiple identities to one implementation.

Example:

old.operation
      |
      v
new.operation

An alias MUST preserve the semantics required by the original contract.

An alias MUST NOT be used to conceal an incompatible API change.

Aliases SHOULD be explicitly declared in provider metadata.


---

33. Redirects

A provider MAY redirect a symbol to another provider.

For example:

Provider A
   |
   v
Redirect
   |
   v
Provider B

Redirects MUST be:

explicit;

bounded;

cycle-detectable;

policy-controlled.


Resolvers MUST detect redirect loops.

Example:

A -> B -> C -> A

MUST fail with a deterministic resolution error.


---

34. Symbol Interposition

Platform-specific symbol interposition MAY exist.

ULABI implementations MUST NOT rely on accidental native linker interposition for semantic correctness.

If interposition is supported, it MUST be explicitly represented by the applicable profile.

Security-sensitive interfaces SHOULD prohibit uncontrolled interposition.


---

35. Multiple Providers

Multiple valid providers may exist.

Example:

Provider A -> CPU
Provider B -> GPU
Provider C -> NPU
Provider D -> Remote

The resolver may select according to explicit policy such as:

required capabilities;

preferred execution target;

latency;

locality;

energy constraints;

availability;

security policy;

deterministic configuration.


Provider selection policy MUST be explicit.


---

36. Provider Priority

A resolver MAY use provider priorities.

Priority MUST NOT override compatibility.

Therefore:

High priority + incompatible

must never beat:

Lower priority + compatible

Compatibility is a prerequisite to ranking.


---

37. Ambiguous Resolution

If multiple candidates remain and no deterministic selection policy applies, resolution MUST fail as ambiguous.

Example:

ULABI-SYMBOL-AMBIGUOUS

Requested:
    org.example.storage.read

Candidates:
    provider.a
    provider.b

The resolver MUST NOT silently choose one.


---

38. Resolution Failure Classes

Implementations SHOULD distinguish at least:

IDENTITY_INVALID
SYMBOL_NOT_FOUND
INTERFACE_NOT_FOUND
VERSION_INCOMPATIBLE
PROFILE_UNSUPPORTED
TYPE_INCOMPATIBLE
CAPABILITY_UNAVAILABLE
SECURITY_POLICY_DENIED
PROVIDER_UNTRUSTED
PROVIDER_INVALID
SYMBOL_AMBIGUOUS
REDIRECT_LOOP
PROVIDER_UNAVAILABLE
LOCALITY_UNSUPPORTED
EXECUTION_MODE_UNSUPPORTED
INTEGRITY_FAILURE

These errors SHOULD be machine-readable.


---

39. Diagnostics

A failed resolution SHOULD provide:

requested identity;

requested version;

requested profiles;

requested capabilities;

candidates discovered;

reasons candidates were rejected;

final failure reason.


Sensitive information MUST NOT be disclosed to unauthorized callers.

Example:

Resolution failed.

Requested:
    org.example.storage.read

Requested version:
    2.x

Rejected candidates:
    provider-a:
        version incompatible

    provider-b:
        capability denied

    provider-c:
        integrity verification failed


---

40. Resolution Cache

Resolvers MAY cache successful resolutions.

A cache entry SHOULD include:

CacheEntry {
    request_identity
    provider_identity
    resolved_version
    profile_set
    capability_context
    integrity_reference
    expiration/invalidation information
}

A cached result MUST NOT outlive the validity of the provider or contract assumptions on which it depends.


---

41. Cache Invalidation

A cached resolution MUST be invalidated when relevant assumptions change.

Examples:

provider unloaded;

provider replaced;

provider revoked;

security policy changes;

capability changes;

interface version changes;

provider integrity changes;

environment changes.


Stale cached bindings MUST NOT silently remain valid.


---

42. Runtime Registry

A runtime MAY maintain a ULABI symbol registry.

Conceptually:

ULABI Registry
 |
 +-- Interface
 +-- Version
 +-- Symbol
 +-- Provider
 +-- Profiles
 +-- Capabilities
 +-- Security Metadata

Registry entries MUST be treated as metadata, not implicit trust.

Registry integrity and authorization remain separate concerns.


---

43. Process-Local Resolution

For in-process interoperability:

Process
 |
 +-- Language A
 |
 +-- ULABI Resolver
 |
 +-- Language B

The resolver MAY bind directly to an implementation address.

However, the implementation MUST still satisfy the ULABI contract.

An address match alone is insufficient.


---

44. Out-of-Process Resolution

For process boundaries:

Process A
   |
   v
ULABI Resolver
   |
   v
Provider Discovery
   |
   v
Process B

Resolution MUST preserve the explicit locality and failure semantics of the interface.

A local symbol MUST NOT silently become an out-of-process call unless the interface permits that execution mode.


---

45. Distributed Resolution

Distributed resolution is governed by:

docs/distributed/distributed-abi.md

docs/distributed/service-discovery.md

docs/distributed/remote-calls.md


This specification only defines the resolution boundary.

The resolver MUST distinguish:

Local Provider
Remote Provider
Unknown Location

A remote provider MUST NOT be treated as equivalent to a local provider when latency, failure, security, or consistency semantics differ.


---

46. Locality Requirements

A symbol reference MAY specify:

LocalOnly
ProcessLocal
HostLocal
NetworkCapable
RemoteCapable

For example:

LocalOnly

MUST NOT resolve to a remote provider.

Likewise:

RemoteCapable

does not require remote execution.


---

47. Hardware Resolution

Hardware and accelerator providers may expose ULABI symbols.

Examples:

CPU
GPU
NPU
FPGA
Quantum
Accelerator

The resolver MAY select an accelerator provider when the requested profile permits it.

Hardware-specific requirements MUST remain explicit.

A generic CPU implementation MUST NOT be selected as a hardware-specific implementation unless graceful degradation is explicitly permitted.


---

48. Quantum and Future Hardware

ULABI symbol resolution MUST remain extensible for future execution targets.

A future provider may expose:

Quantum
Photonic
Neuromorphic
FutureAccelerator

The resolver MUST use registered profile and capability identities rather than hardcoded assumptions about today's hardware.


---

49. Generic Symbols

Generic interfaces MUST use the canonical generic identity defined by the type/interoperability specifications.

A resolver MUST distinguish:

generic declaration

from:

concrete instantiation

when their contracts differ.


---

50. Opaque Symbols

An implementation MAY expose an opaque symbol whose internal representation is intentionally hidden.

Examples:

Handle
Resource
Capability
Context
NativeObject
DeviceContext

The resolver identifies the symbol.

It MUST NOT assume knowledge of the hidden representation.


---

51. Resource Resolution

Resource symbols MUST include the relevant resource contract.

A resolver MUST NOT treat:

resource

as equivalent to:

function

even if both have native addresses.

Resource lifecycle semantics remain governed by the relevant memory/resource specifications.


---

52. Callback Resolution

Callbacks introduce reverse-direction resolution.

Example:

Language A
   |
   | passes callback
   v
ULABI
   |
   v
Language B
   |
   | invokes callback
   v
ULABI
   |
   v
Language A

Callback symbols MUST have stable identities and explicit lifetime rules.

A callback MUST NOT remain callable after its declared lifetime.


---

53. Security Boundary

A resolver MUST treat symbol resolution as a security-sensitive operation when it can determine executable code.

The resolver MUST NOT permit:

arbitrary provider substitution;

unauthorized capability acquisition;

unsigned/untrusted provider execution where policy forbids it;

path-based trust assumptions;

symbol spoofing;

namespace impersonation;

downgrade attacks.



---

54. Namespace Security

Namespace identity is governed by:

docs/interoperability/name-mangling.md

Resolvers SHOULD verify namespace ownership or trust metadata where applicable.

A provider claiming:

org.example.storage

MUST NOT automatically be trusted merely because it uses that namespace.


---

55. Downgrade Protection

A resolver MUST NOT silently downgrade:

requested version = secure/latest

to:

older/insecure version

unless the compatibility and security policies explicitly permit it.

Security policy MUST be able to reject otherwise-compatible older providers.


---

56. Confused Deputy Prevention

A resolver MUST NOT use its own authority to grant capabilities to an untrusted provider merely because another caller requested the provider.

The provider's permissions MUST be evaluated independently.

Conceptually:

Caller Authority
       |
       v
Resolution Request
       |
       v
Provider Authority
       |
       v
Execution Authorization

Caller authority and provider authority MUST remain distinct.


---

57. Trust Is Not Compatibility

These are separate properties:

Compatible
Trusted
Authorized
Available

A provider may be:

Compatible = yes
Trusted = no

Such a provider MUST NOT be selected where trust is required.

Likewise:

Trusted = yes
Compatible = no

must also fail.


---

58. Availability

A provider may be compatible but unavailable.

Examples:

process stopped;

device unavailable;

network unavailable;

resource exhausted;

provider unloaded.


Availability MUST be represented separately from compatibility.


---

59. Recovery

Resolution failure MAY trigger a recovery policy if the applicable reliability profile permits it.

Examples:

Provider A unavailable
        |
        v
Provider B compatible?
        |
       yes
        |
        v
Resolve B

Recovery MUST NOT bypass security or compatibility validation.


---

60. Self-Healing Integration

Self-healing is governed by:

docs/reliability/self-healing.md

Symbol resolution may participate in recovery, but MUST NOT independently modify executable code.

The allowed pattern is:

Failure detected
      |
      v
Evidence
      |
      v
Known resolution policy?
   +--+--+
  yes   no
   |     |
   v     v
Resolve  Escalate
alternative
provider
   |
   v
Validate
   |
   v
Verify

An unknown provider MUST NOT be automatically trusted merely because the original provider failed.


---

61. Deterministic Environments

Safety-critical and deterministic profiles MAY prohibit:

dynamic discovery;

remote resolution;

provider substitution;

lazy resolution;

weak symbols;

uncontrolled aliases.


Such restrictions MUST be profile-defined.


---

62. Real-Time Resolution

A real-time implementation SHOULD avoid unbounded resolution work during a real-time critical section.

A profile MAY require all symbols to be resolved before entering the critical execution phase.


---

63. Embedded Systems

Embedded implementations MAY use:

Compile-Time Registry
Static Symbol Table
ROM Provider Table
Fixed Address Binding

However, the binding MUST still correspond to a valid ULABI contract.


---

64. Resolution Metadata

A conforming provider SHOULD expose machine-readable metadata containing at least:

provider_identity
interface_identity
symbol_identity
contract_identity
version
profiles
capabilities
execution_modes
locality

Security-sensitive implementations SHOULD additionally expose:

integrity_metadata
trust_metadata
signature_metadata

where applicable.


---

65. Canonical Provider Descriptor

Conceptually:

ProviderDescriptor {
    identity: ...
    interfaces: [
        {
            identity: ...
            version: ...
            symbols: [...]
            profiles: [...]
        }
    ]
    capabilities: [...]
    locality: ...
    execution_modes: [...]
}

The exact serialization format is defined by the applicable schema/serialization specification.

This document defines the semantic fields, not the wire encoding.


---

66. Resolution Reference

A reference MAY be represented conceptually as:

ULABIRef {
    namespace
    interface
    symbol
    contract
    version
    profiles
}

The canonical serialized representation MUST be defined by the ULABI schema system.


---

67. Compatibility Matrix

Resolvers SHOULD be able to produce a compatibility matrix:

Candidate	Identity	Version	Profiles	Types	Security	Result

A	✓	✓	✓	✓	✓	Selectable
B	✓	✗	✓	✓	✓	Reject
C	✓	✓	✗	✓	✓	Reject
D	✓	✓	✓	✗	✓	Reject
E	✓	✓	✓	✓	✗	Reject


This is especially important for diagnostics and conformance testing.


---

68. Resolution Invariants

The following invariants are normative.

SR-INV-001 — Identity

A resolved symbol MUST correspond to the requested ULABI identity.

SR-INV-002 — Contract

A resolved provider MUST satisfy the requested contract.

SR-INV-003 — Version

A resolved provider MUST satisfy version compatibility rules.

SR-INV-004 — Profiles

Required profiles MUST be supported.

SR-INV-005 — Capabilities

Required capabilities MUST be authorized.

SR-INV-006 — Security

Security policy MUST be satisfied before executable binding.

SR-INV-007 — Determinism

Ambiguous resolution MUST NOT silently select an arbitrary candidate.

SR-INV-008 — Locality

Execution locality requirements MUST be preserved.

SR-INV-009 — No Implicit Downgrade

Incompatible or forbidden versions MUST NOT be silently selected.

SR-INV-010 — No Identity Spoofing

Native symbols MUST NOT override canonical ULABI identity.

SR-INV-011 — No Unauthorized Substitution

A provider MUST NOT be substituted solely because it has a similar native symbol.

SR-INV-012 — Failure Transparency

Required resolution failures MUST be observable through a defined error mechanism.


---

69. Failure Modes

A resolver MUST account for at least:

InvalidRequest
IdentityNotFound
InterfaceNotFound
SymbolNotFound
VersionMismatch
ProfileMismatch
TypeMismatch
CapabilityDenied
SecurityDenied
ProviderUntrusted
IntegrityFailure
ProviderUnavailable
AmbiguousResolution
RedirectLoop
RegistryFailure
MetadataMalformed
UnsupportedResolutionMode
LocalityViolation


---

70. Recovery Behaviour

Default behavior for required unresolved symbols is:

Resolve
  |
  v
Failure
  |
  v
Apply explicitly authorized recovery policy
  |
  +--> compatible alternative
  |
  +--> fallback
  |
  +--> retry
  |
  +--> escalate

Recovery MUST NOT:

invent a compatible implementation;

silently change contract semantics;

bypass security;

bypass capability checks;

silently change locality;

silently downgrade an interface.



---

71. Compatibility with Future ULABI Versions

A resolver SHOULD be able to reject unknown mandatory fields or profiles safely.

Future extensions MUST be designed so that an older resolver cannot accidentally interpret a new semantic requirement as an old one.

Unknown mandatory requirements MUST produce explicit failure.


---

72. Forward Compatibility

A provider MAY expose additional symbols unknown to a consumer.

Unknown optional symbols MUST generally be ignored.

Unknown mandatory interface requirements MUST cause incompatibility.

This distinction MUST be represented explicitly in provider metadata.


---

73. Backward Compatibility

A newer provider MAY satisfy an older reference if the compatibility rules permit it.

The resolver MUST validate semantic compatibility rather than merely comparing version numbers.


---

74. Symbol Resolution API

A conceptual resolver API is:

resolve(
    SymbolReference,
    ResolutionPolicy
) -> ResolutionResult | ResolutionError

A policy MAY specify:

ResolutionPolicy {
    locality
    security_requirements
    capability_requirements
    preferred_profiles
    provider_preference
    allow_remote
    allow_fallback
    allow_downgrade
    deterministic_only
}

The concrete programming-language API is implementation-specific.


---

75. Binding Lifetime

A successful resolution produces a binding.

The binding MAY be:

DirectAddress
FunctionHandle
InterfaceHandle
ObjectHandle
IPCHandle
ServiceEndpoint
DeviceHandle
RemoteReference

The binding lifetime MUST be explicit.

A binding MUST NOT remain usable after the underlying provider becomes invalid unless the applicable profile explicitly supports rebinding.


---

76. Rebinding

Dynamic systems MAY support rebinding.

Example:

Provider A
   |
   X failure
   |
   v
Provider B

Rebinding MUST revalidate:

identity;

contract;

version;

profiles;

capabilities;

security;

locality.


A previous successful resolution MUST NOT automatically authorize a new provider.


---

77. Concurrency

Resolvers may be called concurrently.

Implementations MUST ensure that concurrent resolution cannot create:

inconsistent provider state;

duplicate unsafe initialization;

capability races;

stale binding races.


The exact synchronization model is implementation-specific.


---

78. Thread Safety

A resolver implementation SHOULD document whether:

resolution is thread-safe;

provider registries are thread-safe;

cache operations are thread-safe;

bindings are thread-safe.


A ULABI profile MAY impose stronger requirements.


---

79. Observability

Resolvers SHOULD expose diagnostics through the ULABI observability system.

Useful events include:

ResolutionRequested
CandidateDiscovered
CandidateRejected
CandidateSelected
ResolutionFailed
BindingCreated
BindingInvalidated
ProviderRevoked
RebindingStarted
RebindingCompleted

Sensitive metadata MUST be redacted according to security policy.


---

80. Performance

Resolvers SHOULD support:

indexed lookup;

cached metadata;

precomputed identities;

static resolution;

lazy resolution where allowed;

batch resolution.


Optimization MUST NOT change semantic resolution results.


---

81. Batch Resolution

Implementations MAY resolve multiple symbols together.

Example:

resolve_many([
    symbol_a,
    symbol_b,
    symbol_c
])

Batch resolution MAY optimize provider discovery and validation.

Each symbol MUST still satisfy its individual contract.


---

82. Atomic Interface Resolution

An interface MAY require atomic resolution.

Example:

Interface Storage v2
    |
    +-- open
    +-- read
    +-- write
    +-- close

If the interface declares atomic binding, the implementation MUST NOT expose a partially resolved interface.


---

83. Partial Interfaces

Partial resolution is permitted only when the interface contract explicitly permits optional operations.

Required operations MUST be resolved before the interface is considered valid.


---

84. Cross-Language Example

Language A exports:

function calculate(x: Int32) -> Int32

Language B requests:

org.example.math.calculate

The resolver performs:

Request
   |
   v
Identity lookup
   |
   v
Provider discovered
   |
   v
Contract validation
   |
   +-- parameter = Int32 ✓
   +-- return = Int32 ✓
   +-- version ✓
   +-- profile ✓
   +-- security ✓
   |
   v
Resolved

The source-level function name is irrelevant after identity binding.


---

85. Multiple Language Implementations

The same ULABI interface may be implemented by:

C provider
Rust provider
Go provider
Python provider
Swift provider
Kotlin provider
Zamani provider
Sankofa provider

The resolver does not prefer a language.

Selection is based on the resolution policy and provider compatibility.


---

86. Implementation Independence

A resolver MUST NOT assume:

C ABI;

C++ ABI;

Rust ABI;

JVM ABI;

Python ABI;

CLR ABI;

a particular garbage collector;

a particular object model.


Native implementation details belong below the ULABI boundary.


---

87. Security Requirements Summary

A conforming secure resolver:

1. validates canonical identity;


2. validates provider metadata;


3. validates compatibility;


4. validates required capabilities;


5. validates security policy;


6. validates provider integrity where required;


7. prevents unauthorized substitution;


8. prevents downgrade attacks;


9. detects redirect loops;


10. protects against malformed metadata;


11. separates trust from compatibility;


12. records security-relevant diagnostics.




---

88. Conformance Requirements

A symbol resolver claiming conformance MUST demonstrate:

SR-C-001

Correct canonical symbol identity handling.

SR-C-002

Correct interface matching.

SR-C-003

Correct contract matching.

SR-C-004

Correct version matching.

SR-C-005

Correct profile matching.

SR-C-006

Correct capability handling.

SR-C-007

Correct security-policy enforcement.

SR-C-008

Deterministic ambiguity handling.

SR-C-009

Correct optional/required symbol behavior.

SR-C-010

Correct alias and redirect handling.

SR-C-011

Redirect-loop detection.

SR-C-012

Correct failure reporting.

SR-C-013

Correct cache invalidation.

SR-C-014

Correct locality enforcement.

SR-C-015

Correct rebinding validation where supported.


---

89. Required Conformance Test Categories

The future conformance suite MUST include:

identity/
contract/
version/
profiles/
capabilities/
security/
ambiguity/
aliases/
redirects/
optional/
weak/
static/
dynamic/
lazy/
eager/
locality/
cache/
invalidation/
rebinding/
failure/
diagnostics/
cross-language/

Representative tests:

symbol_identity_exact
symbol_identity_mismatch
interface_identity_mismatch
contract_mismatch
version_compatible
version_incompatible
required_profile_missing
optional_profile_missing
capability_denied
untrusted_provider
ambiguous_provider
redirect_loop
missing_symbol
optional_symbol_missing
local_only_rejects_remote
cache_invalidated_after_provider_removal
rebind_requires_revalidation


---

90. Reference Implementation Requirements

A future reference resolver SHOULD provide:

Resolver
 |
 +-- IdentityValidator
 +-- CandidateDiscovery
 +-- ContractValidator
 +-- VersionMatcher
 +-- ProfileMatcher
 +-- CapabilityValidator
 +-- SecurityValidator
 +-- ProviderSelector
 +-- AmbiguityDetector
 +-- BindingManager
 +-- CacheManager
 +-- InvalidationManager
 +-- Diagnostics

These are conceptual implementation modules.

They MAY be combined in a small implementation, but their responsibilities MUST remain distinguishable.


---

91. Required Code/Module Architecture

A production implementation SHOULD eventually provide modules equivalent to:

symbol/
├── identity
├── reference
├── descriptor
├── resolver
├── discovery
├── candidate
├── matcher
├── contract
├── version
├── profile
├── capability
├── security
├── provider
├── selector
├── ambiguity
├── alias
├── redirect
├── binding
├── cache
├── invalidation
├── rebinding
├── locality
├── registry
├── diagnostics
└── errors

These names are implementation-neutral. They are not mandated as filenames by the ULABI specification.


---

92. Integration Contract

This specification integrates with the repository as follows.

Upstream

The resolver consumes:

ULABI-DESIGN.md
ULABI-SPEC.md
ULABI-VERSIONING.md

for architectural, normative, and versioning rules.

Identity

It consumes:

docs/interoperability/name-mangling.md

for canonical symbol identity.

Language layer

It integrates with:

docs/interoperability/language-interoperability.md

for language adapter semantics.

FFI

It passes resolved bindings to:

docs/interoperability/foreign-function-interface.md

for invocation.

Data

It relies on:

docs/interoperability/cross-language-data.md

for cross-language value contracts.

Objects

It relies on:

docs/interoperability/object-model.md

for object/interface semantics.

ABI

It relies on:

docs/abi/core-abi.md
docs/abi/calling-convention.md
docs/abi/data-types.md
docs/abi/exception-model.md

for ABI semantics.

Compatibility

It integrates with:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Security

It integrates with:

docs/security/security-model.md
docs/security/secure-loading.md

Distributed operation

It integrates with:

docs/distributed/distributed-abi.md
docs/distributed/service-discovery.md
docs/distributed/remote-calls.md

Reliability

It integrates with:

docs/reliability/self-healing.md
docs/reliability/recovery.md
docs/reliability/rollback.md

Standards

It integrates with:

docs/standards/conformance.md
docs/standards/test-suite.md
docs/standards/certification.md


---

93. No Circular Authority

This specification MUST remain a consumer of the other specifications.

It MUST NOT redefine:

canonical type identity;

calling conventions;

memory ownership;

serialization;

security primitives;

transport;

language syntax.


The architecture is:

ULABI-DESIGN
      |
ULABI-SPEC
      |
      +------------------------------+
      |                              |
Identity Specifications       ABI Specifications
      |                              |
      +---------------+--------------+
                      |
              Symbol Resolution
                      |
             +--------+--------+
             |                 |
            FFI          Runtime/Loader


---

94. Implementation Independence

Different implementations may organize their resolver differently.

For example:

Implementation A:
single resolver module

Implementation B:
distributed resolver services

Implementation C:
compile-time resolver

Implementation D:
runtime resolver

Implementation E:
embedded static resolver

All are valid if they satisfy the same ULABI contract.


---

95. Formal Resolution Property

For a resolution request R and provider set P:

Resolve(R, P) = S

is valid only if:

Identity(S) = Identity(R)

AND

Contract(S) satisfies Contract(R)

AND

Version(S) is compatible with Version(R)

AND

Profiles(S) satisfy RequiredProfiles(R)

AND

Capabilities(S) satisfy RequiredCapabilities(R)

AND

SecurityPolicy(S, R) = allowed

AND

Locality(S) satisfies Locality(R)

AND

ExecutionMode(S) satisfies ExecutionRequirements(R)

Otherwise:

Resolve(R, P) = Failure

A resolver MUST NOT return an invalid candidate as a successful resolution.


---

96. Final Architectural Rule

ULABI symbol resolution is therefore:

Identity
   ↓
Discovery
   ↓
Validation
   ↓
Compatibility
   ↓
Authorization
   ↓
Selection
   ↓
Binding
   ↓
Execution

The most important invariant is:

> Finding a symbol is not the same as resolving a ULABI contract.



ULABI resolution is successful only when the selected implementation is demonstrably compatible with the requested contract and authorized for the requested execution context.


---

97. Completion Criteria

docs/interoperability/symbol-resolution.md is considered complete for this specification phase when:

canonical identity is consumed from name-mangling.md;

symbol references are defined;

provider descriptors are defined;

discovery and resolution are separated;

contract validation is defined;

version matching is delegated correctly;

profile matching is defined;

capability/security checks are defined;

ambiguity behavior is defined;

optional and weak symbols are defined;

aliases and redirects are defined;

static/dynamic/lazy/eager resolution are defined;

local/process/distributed resolution boundaries are defined;

cache and invalidation semantics are defined;

rebinding semantics are defined;

failure modes are defined;

conformance requirements are defined;

integration points are explicitly documented;

no language-specific implementation is required;

no Zamani- or Sankofa-specific behavior is required.


No later document should need to redefine this symbol-resolution contract.

## What should be done under which files

The important thing is **not to make `symbol-resolution.md` responsible for everything around it**. I would lock the ownership boundaries now.

### Phase 1 — independent foundations

These should be completed first because other specifications consume them.

| Order | File | Responsibility |
|---|---|---|
| 1 | `ULABI-VERSIONING.md` | Version identity, compatibility classes, evolution rules |
| 2 | `docs/abi/data-types.md` | Canonical ABI type identities |
| 3 | `docs/interoperability/name-mangling.md` | Canonical symbol/interface identity |
| 4 | `docs/compatibility/backwards-compatibility.md` | Older consumer/newer provider rules |
| 5 | `docs/compatibility/forwards-compatibility.md` | Newer consumer/older provider rules |
| 6 | `docs/compatibility/capability-discovery.md` | Capability advertisement/discovery semantics |
| 7 | `docs/security/security-model.md` | Trust, authorization, security boundaries |

`name-mangling.md` is already substantially developed in the repository and should remain the authority for identity rather than being duplicated here. 

### Phase 2 — direct symbol-resolution dependencies

| File | What belongs there |
|---|---|
| `docs/interoperability/symbol-resolution.md` | **This document** — resolving a ULABI identity to an implementation |
| `docs/interoperability/language-interoperability.md` | Language adapters and language-to-ULABI mapping |
| `docs/interoperability/foreign-function-interface.md` | How a resolved symbol is actually invoked |
| `docs/interoperability/object-model.md` | Object/interface/dispatch semantics |
| `docs/interoperability/cross-language-data.md` | Value/data conversion |

The existing language-interoperability document already correctly positions itself as an integration layer rather than a replacement for FFI, cross-language data, object model, or symbol resolution. 

### Phase 3 — ABI/runtime integration

| File | Responsibility |
|---|---|
| `docs/abi/core-abi.md` | Core binary contract |
| `docs/abi/calling-convention.md` | Actual call lowering |
| `docs/abi/memory-model.md` | Boundary memory semantics |
| `docs/abi/exception-model.md` | ABI-level failures |
| `docs/runtime/runtime-interface.md` | Runtime services |
| `docs/runtime/resource-management.md` | Resource lifetime |
| `docs/tooling/linker-interface.md` | Linker integration |
| `docs/tooling/loader-interface.md` | Loading integration |
| `docs/tooling/validator.md` | Contract validation |

### Phase 4 — security and dynamic loading

| File | Responsibility |
|---|---|
| `docs/security/secure-loading.md` | Safe loading of resolved providers |
| `docs/security/capability-security.md` | Capability enforcement |
| `docs/security/sandboxing.md` | Isolation |
| `docs/security/supply-chain-security.md` | Provider/artifact provenance |
| `docs/security/authentication.md` | Provider identity authentication |
| `docs/security/authorization.md` | Permission decisions |

### Phase 5 — distributed resolution

| File | Responsibility |
|---|---|
| `docs/distributed/distributed-abi.md` | Distributed ABI boundary |
| `docs/distributed/service-discovery.md` | Discovering remote providers |
| `docs/distributed/remote-calls.md` | Invoking remote symbols |
| `docs/distributed/serialization.md` | Remote representation |
| `docs/distributed/distributed-errors.md` | Remote failure semantics |
| `docs/distributed/consensus-boundaries.md` | Distributed agreement boundaries |

### Phase 6 — reliability

| File | Responsibility |
|---|---|
| `docs/reliability/fault-detection.md` | Detect provider/resolution faults |
| `docs/reliability/fault-isolation.md` | Isolate failed providers |
| `docs/reliability/recovery.md` | Recovery policy |
| `docs/reliability/rollback.md` | Safe rollback |
| `docs/reliability/self-healing.md` | Complete bounded recovery orchestration |
| `docs/reliability/health-monitoring.md` | Provider health |

The important boundary is that `symbol-resolution.md` can **request** a recovery/rebinding policy but must never define autonomous code modification. That remains with the reliability subsystem.

---

# Required code/module architecture

The repository is currently primarily specification/design-oriented, so I would **not create implementation-specific Rust/Sankofa/etc. code inside the standard itself yet**.

Instead, reserve the following language-neutral implementation architecture:

```text
implementations/
└── symbol-resolution/
    ├── identity/
    │   ├── canonical_identity
    │   └── identity_validator
    │
    ├── reference/
    │   └── symbol_reference
    │
    ├── descriptor/
    │   └── provider_descriptor
    │
    ├── discovery/
    │   ├── static_discovery
    │   ├── runtime_discovery
    │   └── registry_discovery
    │
    ├── matching/
    │   ├── contract_matcher
    │   ├── version_matcher
    │   ├── profile_matcher
    │   ├── type_matcher
    │   └── capability_matcher
    │
    ├── security/
    │   ├── trust_validator
    │   ├── integrity_validator
    │   └── authorization_validator
    │
    ├── resolution/
    │   ├── resolver
    │   ├── selector
    │   └── ambiguity_detector
    │
    ├── binding/
    │   ├── binding_manager
    │   ├── binding_lifetime
    │   └── rebinding
    │
    ├── registry/
    │   └── provider_registry
    │
    ├── cache/
    │   ├── resolution_cache
    │   └── cache_invalidation
    │
    ├── aliases/
    │   └── alias_resolver
    │
    ├── redirects/
    │   └── redirect_resolver
    │
    ├── locality/
    │   └── locality_policy
    │
    ├── diagnostics/
    │   └── resolution_diagnostics
    │
    └── errors/
        └── resolution_errors

These are responsibility names, not a mandate that ULABI implementations use exactly these filenames.

Conformance/test modules

The corresponding future test architecture should be:

tests/
└── symbol-resolution/
    ├── identity/
    ├── interface/
    ├── contract/
    ├── version/
    ├── profiles/
    ├── capabilities/
    ├── security/
    ├── ambiguity/
    ├── aliases/
    ├── redirects/
    ├── optional-symbols/
    ├── weak-symbols/
    ├── static-resolution/
    ├── dynamic-resolution/
    ├── lazy-resolution/
    ├── eager-resolution/
    ├── locality/
    ├── caching/
    ├── invalidation/
    ├── rebinding/
    ├── distributed/
    ├── diagnostics/
    └── failure-modes/

And the conformance suite should eventually expose tests such as:

SR-C-001 canonical identity
SR-C-002 interface matching
SR-C-003 contract matching
SR-C-004 version compatibility
SR-C-005 profile compatibility
SR-C-006 capability enforcement
SR-C-007 security enforcement
SR-C-008 ambiguity detection
SR-C-009 optional symbol handling
SR-C-010 alias handling
SR-C-011 redirect-loop detection
SR-C-012 resolution failure reporting
SR-C-013 cache invalidation
SR-C-014 locality enforcement
SR-C-015 safe rebinding

Critical architectural decision

The final dependency direction should be:

ULABI-DESIGN
                       │
                   ULABI-SPEC
                       │
          ┌────────────┴────────────┐
          │                         │
   Identity contracts        ABI contracts
          │                         │
          └────────────┬────────────┘
                       │
                SYMBOL RESOLUTION
                       │
          ┌────────────┼────────────┐
          │            │            │
         FFI       Runtime/Loader  Security
          │            │            │
          └────────────┼────────────┘
                       │
              Concrete Provider

This prevents the common architectural mistake where the linker, FFI, runtime, or one programming language becomes the hidden owner of ULABI.

Also, do not make symbol-resolution.md depend on Zamani or Sankofa. Both remain merely possible providers/consumers, exactly as the repository's master architecture requires.

Current repository status relevant to this work

ULABI-DESIGN.md: substantive architecture, currently draft 0.2.0.

docs/interoperability/name-mangling.md: already substantially specified; it correctly separates semantic identity from native linkage.

docs/interoperability/symbol-resolution.md: currently only a placeholder, so the document above can replace it directly.

docs/interoperability/language-interoperability.md: already has the correct high-level responsibility boundary and should consume, not duplicate, symbol resolution.

The repository already contains separate ABI, compatibility, architecture, and distributed specifications, which is the right direction for avoiding a monolithic standard.


One important rule for the next documents: once this file is accepted, later files should reference its SR-INV-*, SR-C-*, data structures, and resolution lifecycle rather than redefining symbol resolution. That is how we achieve your requirement that a completed file does not have to be reopened merely because another component was subsequently written.