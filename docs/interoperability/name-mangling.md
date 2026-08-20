I inspected the ULABI repository before writing this. The current ULABI-DESIGN.md is the architectural authority and explicitly establishes ULABI as language-, runtime-, OS-, CPU-, and vendor-neutral, with a Minimal Core + Standard Profiles + Extensible Ecosystem model.

I also checked the existing interoperability/ABI structure. language-interoperability.md and name-mangling.md are currently placeholders, while the repository already has separate documents for the core ABI, calling convention, data types, memory model, distributed ABI, compatibility negotiation/discovery, and related areas.

So name-mangling.md should not redefine types, calling conventions, serialization, FFI, or language interoperability. Its job is specifically to define how a source-level declaration becomes a stable, language-neutral linkage identity.

Below is the complete document to copy into:

docs/interoperability/name-mangling.md


---

ULABI Name Mangling and Symbol Identity Specification

Document: docs/interoperability/name-mangling.md
Project: ULABI — Universal Language Application Binary Interface
Status: Draft Normative Specification
Specification Family: Interoperability
Layer: Interface Identity / Symbol Resolution
Authority: ULABI-SPEC.md + ULABI-DESIGN.md
Language Dependency: None
Runtime Dependency: None
Operating-System Dependency: None
CPU Architecture Dependency: None


---

1. Purpose

This specification defines how ULABI assigns stable identities to externally visible functions, methods, types, constants, variables, interfaces, resources, and other linkable entities.

The purpose is to allow independently implemented languages, compilers, linkers, runtimes, loaders, libraries, and tools to identify the same ULABI contract without requiring them to share:

source syntax;

naming conventions;

compiler implementation;

runtime;

operating system;

CPU architecture;

object-file format;

linker;

programming language;

vendor.


ULABI name mangling is therefore primarily an interface identity mechanism, not a compiler-specific decoration mechanism.


---

2. Fundamental Principle

ULABI distinguishes three different concepts:

Source Name
     |
     v
Language-Level Declaration
     |
     v
ULABI Interface Identity
     |
     v
Transport / Object / Linker Symbol

These must not be conflated.

A language may internally call a function:

calculate

Another language may call the equivalent operation:

bereken

Another may represent it through a method:

Calculator.calculate

ULABI does not require these source names to match.

Instead, both implementations must explicitly bind their declarations to the same ULABI interface identity when they implement the same contract.

Therefore:

> ULABI identity is semantic and contract-based, not source-name-based.




---

3. Scope

This specification defines:

1. symbol identity;


2. canonical interface identifiers;


3. namespace identity;


4. declaration identity;


5. operation identity;


6. overload identity;


7. method identity;


8. generic identity;


9. version identity;


10. ABI-profile identity;


11. symbol encoding;


12. canonicalization;


13. escaping;


14. Unicode handling;


15. collision prevention;


16. symbol lookup;


17. aliases;


18. external bindings;


19. dynamic symbol discovery;


20. symbol compatibility;


21. security requirements;


22. diagnostics;


23. conformance requirements.



This specification does not define:

function calling conventions;

parameter passing;

memory layout;

serialization formats;

object layout;

language syntax;

language-specific type systems;

network transport;

executable file formats.


Those responsibilities belong to their respective ULABI specifications.


---

4. Terminology

4.1 Source Name

The name assigned by a programming language.

Example:

calculate


---

4.2 Declaration

A language-level definition that can be exposed through ULABI.

Examples:

function
method
type
constant
variable
interface
resource
service


---

4.3 Symbol

A linkable or discoverable entity associated with a ULABI interface.


---

4.4 ULABI Interface Identity

A globally meaningful identifier representing a specific ULABI contract.


---

4.5 Symbol Identity

The identity used to distinguish one exported entity from another.


---

4.6 Canonical Identity

The normalized representation from which the stable ULABI identity is derived.


---

4.7 Mangled Symbol

An encoded symbol representation suitable for a particular linkage environment.


---

4.8 Linkage Name

The name used by an object format, linker, loader, runtime registry, or foreign-function interface.


---

4.9 Alias

An additional name referring to an existing symbol identity.


---

4.10 Namespace

A logical scope used to prevent unrelated interfaces from colliding.


---

5. Design Requirements

ULABI name mangling MUST satisfy the following requirements.

NMR-001 — Language neutrality

The canonical identity MUST NOT depend on a particular programming language.

NMR-002 — Compiler neutrality

The identity MUST NOT depend on compiler implementation details.

NMR-003 — Runtime neutrality

A runtime MUST NOT be required to understand a particular language's mangling scheme.

NMR-004 — Determinism

The same canonical declaration MUST always produce the same canonical identity.

NMR-005 — Collision resistance

Distinct ULABI identities MUST NOT intentionally map to the same canonical identity.

NMR-006 — Explicit semantics

A symbol's identity MUST be derived from explicitly defined semantic information.

NMR-007 — Version awareness

Changing a contract in a compatibility-breaking way MUST produce a distinguishable identity.

NMR-008 — Architecture independence

CPU-specific properties MUST NOT be embedded in the Core identity unless explicitly represented as a profile requirement.

NMR-009 — Human inspectability

Implementations SHOULD provide a human-readable representation for diagnostics.

NMR-010 — Machine stability

Human-readable source names MUST NOT be relied upon as the sole identity mechanism.


---

6. Source Names Are Not ULABI Identities

Consider:

Language A:

function add(a: Int32, b: Int32) -> Int32

and:

Language B:

fn plus(x: Integer, y: Integer) -> Integer

The source names differ.

ULABI does not assume:

add == plus

Instead, both implementations can declare:

ULABI Interface:
    org.example.math
    operation: addition
    contract: <stable contract identity>

The ULABI identity is therefore independent of the local spelling.


---

7. Canonical Symbol Identity

A ULABI symbol MUST have a canonical semantic identity.

Conceptually:

SymbolIdentity =
    Namespace
    +
    EntityKind
    +
    EntityIdentifier
    +
    ContractIdentity
    +
    Version
    +
    ProfileRequirements

The canonical identity MUST be defined independently of the target linker.


---

8. Entity Kinds

ULABI MUST distinguish at least:

function
method
constructor
destructor
type
constant
variable
interface
resource
service
capability
namespace
module

Additional entity kinds may be introduced through registered extensions.

Unknown entity kinds MUST NOT silently be interpreted as another entity kind.


---

9. Namespaces

Namespaces prevent unrelated components from accidentally sharing identities.

A namespace SHOULD use a globally controlled identifier.

Examples:

org.example
com.vendor.project
dev.library

A namespace MUST NOT rely solely on a programming-language package name.

For example:

package foo

does not automatically establish a globally unique ULABI namespace.


---

10. Namespace Ownership

Namespace allocation SHOULD be governed independently of language implementations.

A namespace MAY be associated with:

an organization;

an open-source project;

an individual;

a standards organization;

an ecosystem registry.


ULABI does not require a centralized namespace registry for Core operation, but implementations SHOULD support registry-backed namespace verification where available.


---

11. Entity Identifiers

An entity identifier MUST be stable.

For example:

org.example.math.add

may identify an operation.

However, the textual path alone is insufficient if the contract changes incompatibly.

Therefore the identity also includes contract/version information.


---

12. Function Identity

A function identity MUST distinguish at minimum:

namespace
entity kind
operation identity
contract identity
version

Example conceptual identity:

namespace = org.example.math
kind = function
operation = add
contract = ...
version = 1


---

13. Overloaded Functions

ULABI MUST NOT rely on source-language overload rules.

For example:

add(Int32, Int32)
add(Float64, Float64)

MUST produce distinct identities.

The distinction MUST derive from the ULABI contract.

Conceptually:

add
 ├── contract A
 └── contract B

The canonical identity MUST therefore contain enough information to distinguish the two operations.


---

14. Parameter Types

Parameter types MUST be represented using ULABI type identities.

ULABI name mangling MUST NOT directly encode arbitrary language-specific type spellings.

For example, these are not canonical:

std::string
String
java.lang.String
str
string

Instead they MUST resolve to a ULABI type identity.


---

15. Return Types

Return types MUST NOT be included merely because a source language uses return-type-based overloading.

ULABI should distinguish operations according to the contract rules defined by the function ABI and type system.

Where return type participates in contract identity, it MUST use the canonical ULABI type identity.

This prevents language-specific overload behavior from leaking into ULABI.


---

16. Generic Functions

Generic declarations MUST have stable generic identities.

Example:

map<T, U>(...)

must not depend on the source syntax used to express generics.

A generic contract SHOULD identify:

generic parameter count
generic parameter constraints
generic variance
generic contract identity

where those properties affect interoperability.


---

17. Generic Instantiations

A concrete generic instantiation MAY have a derived identity.

Example:

map<Int32, String>

However, the identity MUST remain distinguishable from:

map<Float64, String>

ULABI implementations MUST NOT assume that generic instantiations have identical binary representations.


---

18. Methods

Methods MUST include their containing contract/type identity.

Conceptually:

TypeIdentity
+
MethodIdentity
+
ContractIdentity

This prevents:

A.calculate

from colliding with:

B.calculate

even when both use the same source-level name.


---

19. Constructors

Constructors MUST have explicit identities.

A constructor identity SHOULD include:

constructed type
constructor identity
parameter contract
version


---

20. Destructors and Finalizers

Destructors, finalizers, or equivalent resource-release operations MUST NOT be assumed to be interchangeable across languages.

If exposed through ULABI, their contract MUST explicitly define:

ownership;

lifetime;

invocation conditions;

failure behavior;

idempotency;

resource semantics.


The name-mangling layer identifies the operation but does not define its resource semantics.


---

21. Constants and Variables

Exported constants and variables MUST have distinct entity kinds.

For example:

function:add
constant:add
variable:add

MUST NOT collide.


---

22. Interface Identity

An interface MUST have an independent identity from individual operations.

Conceptually:

Interface:
    org.example.storage

Operations:
    open
    read
    write
    close

The interface identity provides the stable contract boundary.


---

23. Module Identity

Modules MAY expose multiple interfaces.

A module identity MUST NOT automatically become the identity of every contained symbol.

Therefore:

module A
    function x
    function y

does not imply:

A == x
A == y

Each entity retains its own identity.


---

24. Contract Identity

A symbol MUST ultimately identify a contract rather than merely a spelling.

A contract includes the relevant semantic properties defined by other ULABI specifications.

Depending on the entity, these may include:

parameter contracts;

return contracts;

effects;

ownership;

capabilities;

execution semantics;

version;

profile requirements.


The name-mangling specification references these properties but does not redefine them.


---

25. Canonicalization

Before identity generation, canonical semantic data MUST be normalized.

Canonicalization MUST be deterministic.

Equivalent canonical declarations MUST produce identical canonical input.

Non-equivalent declarations MUST remain distinguishable.


---

26. Unicode

ULABI supports Unicode identifiers.

However, unrestricted Unicode can introduce:

visually identical identifiers;

normalization ambiguity;

confusable characters;

bidirectional-control attacks;

security-sensitive spoofing.


Therefore ULABI implementations MUST distinguish:

semantic identifier

from:

display identifier


---

27. Unicode Normalization

ULABI canonical identifiers MUST use one explicitly specified Unicode normalization form.

The selected normalization form MUST be applied before identity generation.

An implementation MUST NOT silently use different normalization forms depending on host operating system or programming language.


---

28. Unicode Confusables

Implementations SHOULD detect confusable identifiers.

For example:

calculate
cаlculate

where one character may originate from a different Unicode script.

Confusable detection is a diagnostic/security facility.

It MUST NOT silently merge distinct identifiers.


---

29. Bidirectional Controls

ULABI symbol identifiers MUST reject or explicitly escape dangerous bidirectional control characters in canonical identity material.

A display system MUST NOT render a symbol in a way that changes its apparent semantic ordering.


---

30. Case Sensitivity

ULABI canonical identities MUST be case-sensitive unless a specific namespace profile explicitly defines otherwise.

Therefore:

Calculate

and:

calculate

are distinct identities.

A case-insensitive environment MUST NOT silently merge them.


---

31. Reserved Characters

The canonical textual representation MUST define escaping for characters used as structural delimiters.

For example:

:
/
.
@
#
%
;

may have structural meaning.

Literal occurrences MUST be escaped or encoded.


---

32. Canonical Encoding

ULABI should define a canonical machine-readable encoding separate from human-readable symbols.

Conceptually:

Canonical Symbol Record
        |
        +-- namespace
        +-- entity kind
        +-- identifier
        +-- contract
        +-- version
        +-- profiles
        +-- attributes

The canonical record becomes the source of truth.


---

33. Human-Readable Representation

A human-readable representation SHOULD be available for:

debugging;

diagnostics;

linker errors;

ABI inspection;

documentation;

tooling.


It MUST NOT replace the canonical identity.


---

34. Machine Linkage Names

A platform-specific linker may impose restrictions on symbol names.

Therefore:

ULABI Identity
       |
       v
Linkage Encoding
       |
       v
Platform Symbol

A platform linkage name is an encoding of ULABI identity.

It is not itself the normative identity.


---

35. Platform Symbol Encoding

An implementation MAY encode a ULABI symbol into:

ELF symbols;

Mach-O symbols;

PE/COFF symbols;

WebAssembly exports;

runtime registries;

shared-library exports;

static-link tables;

custom object formats.


The underlying format MUST NOT change the ULABI identity.


---

36. Architecture Independence

A ULABI identity MUST NOT change solely because an implementation moves from:

x86-64

to:

ARM64

or:

RISC-V

unless the interface contract explicitly requires an architecture-specific profile.


---

37. Architecture-Specific Profiles

An interface MAY declare architecture-specific requirements.

For example:

profile = hardware.vector
architecture = ...

Such requirements MUST be explicit.

Architecture information MUST NOT be hidden inside ordinary symbol spelling.


---

38. ABI Profile Identity

Where an interface requires a particular ABI profile, the profile identity MUST be represented separately from the basic symbol identity.

For example:

Core
Core + Memory
Core + GPU
Core + Tensor

must remain distinguishable through profile metadata.


---

39. Versioning

Symbol identity MUST integrate with ULABI versioning.

A compatible evolution MAY retain an existing identity.

An incompatible contract change MUST produce a distinguishable contract/version identity.

The exact compatibility rules are governed by:

ULABI-VERSIONING.md
docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md

The name-mangling layer MUST consume those rules rather than redefine them.


---

40. ABI Version Versus Interface Version

These are different:

ULABI specification version

and:

interface contract version

A change to the ULABI specification does not automatically mean every application interface changes identity.

Likewise, an interface can evolve while the ULABI specification remains unchanged.


---

41. Aliases

ULABI MAY support aliases.

Example:

old-name -> stable-interface-id

Aliases MUST NOT create a second semantic contract.

They only provide another resolution path to an existing identity.


---

42. Alias Security

Aliases MUST be explicitly declared.

An implementation MUST NOT accept arbitrary aliases supplied by untrusted input without appropriate authorization.

Otherwise attackers could redirect an expected symbol to an unintended implementation.


---

43. Symbol Resolution

Symbol resolution SHOULD follow:

Requested Identity
       |
       v
Namespace Resolution
       |
       v
Entity Resolution
       |
       v
Contract Verification
       |
       v
Version Compatibility
       |
       v
Profile Compatibility
       |
       v
Binding

A matching textual name alone is insufficient.


---

44. Resolution Failure

If a symbol cannot be resolved, the implementation MUST return an explicit failure.

Possible failure classes include:

NamespaceNotFound
SymbolNotFound
AmbiguousSymbol
ContractMismatch
VersionMismatch
ProfileUnsupported
TypeContractMismatch
SecurityPolicyDenied
InvalidIdentity
MalformedEncoding

These errors should integrate with the ULABI exception/error model rather than creating an unrelated error mechanism.


---

45. Ambiguous Symbols

An implementation MUST NOT arbitrarily select one symbol when multiple candidates satisfy a non-unique lookup.

It MUST instead report:

AmbiguousSymbol

unless an explicit deterministic resolution policy exists.


---

46. Symbol Discovery

Symbol discovery MAY expose:

namespace
entity kind
symbol identity
contract identity
version
profiles
capabilities
visibility

Discovery MUST NOT imply that the caller is authorized to invoke the symbol.

Discovery and authorization are separate concerns.


---

47. Visibility

ULABI symbols MAY be:

Private
Module
Package
Interface
Public
System

The exact visibility model belongs to the exposing environment.

ULABI identity MUST remain independent from language-specific visibility syntax.


---

48. Symbol Export

An implementation exporting a symbol MUST provide enough metadata for another implementation to determine:

1. identity;


2. contract;


3. version;


4. required profiles;


5. applicable compatibility rules.




---

49. Symbol Import

An importer MUST verify the expected identity before binding.

An importer MUST NOT assume that:

same textual name

means:

same contract


---

50. Weak and Optional Symbols

Optional symbols MAY be represented through explicit availability metadata.

A missing optional symbol MUST NOT be treated as a successful binding.

The compatibility system may then determine whether graceful degradation is possible.


---

51. Symbol Interposition

Platform-specific symbol interposition MUST NOT silently alter ULABI contract identity.

If an implementation intentionally redirects an interface, that redirection MUST be visible to the applicable security and observability mechanisms.


---

52. Dynamic Loading

Dynamic loaders MAY resolve ULABI symbols at runtime.

The resolution sequence SHOULD be:

Load Artifact
     |
Validate Identity Metadata
     |
Validate Security
     |
Resolve ULABI Identity
     |
Validate Contract
     |
Validate Version
     |
Validate Profiles
     |
Bind


---

53. Static Linking

Static linkers MAY resolve ULABI identities at build time.

They MUST preserve enough metadata for diagnostics and compatibility analysis.


---

54. Link-Time Optimization

Link-time optimization MUST NOT change the semantic identity of an externally visible ULABI contract.

An optimizer may change implementation details.

It must preserve the exposed contract.


---

55. Symbol Elimination

An implementation MUST NOT eliminate an externally required symbol merely because the implementation considers it unused internally.

If the symbol is part of an exported ULABI contract, its externally observable semantics remain authoritative.


---

56. Compiler Independence

A compiler MAY choose any internal representation.

For example:

AST
IR
SSA
bytecode
machine code
JIT representation
interpreter representation

None of these are part of ULABI identity.


---

57. Language Independence

The following source-language concepts MUST NOT become mandatory ULABI concepts merely because they are common:

C++ namespaces
Rust modules
Java packages
Python modules
C# namespaces
Swift modules
Zamani modules
Sankofa modules

Each language must provide an adapter from its own naming model to ULABI identity.


---

58. Multiple Language Bindings

Multiple source declarations MAY bind to one ULABI interface if they implement the same contract.

Example:

Language A: calculate
Language B: compute
Language C: evaluate

All may bind to:

ULABI Interface Identity: <stable identity>

provided their contracts are compatible.


---

59. One Source Declaration, Multiple Interfaces

A source declaration MAY implement multiple ULABI interfaces.

For example:

calculate
   |
   +-- Math.Basic.calculate
   |
   +-- Scientific.Calculation.calculate

Each binding MUST be explicitly declared.


---

60. Name Mangling Must Not Create Semantics

Name mangling identifies a contract.

It MUST NOT be used to secretly encode:

ownership behavior;

security permissions;

memory safety;

execution effects;

authorization;

network behavior.


Those properties must be represented through their respective ULABI metadata.


---

61. Security Boundary

A symbol name MUST NOT be treated as an authorization credential.

Knowing:

org.example.admin.shutdown

does not grant permission to invoke it.

Authorization belongs to the ULABI security model.


---

62. Symbol Spoofing

Implementations MUST defend against symbol spoofing.

Threats include:

Unicode confusables;

malicious aliases;

namespace impersonation;

version confusion;

profile confusion;

truncation;

encoding ambiguity;

case folding;

delimiter injection.



---

63. Canonical Comparison

Symbol identities MUST be compared using canonical machine representations.

Implementations MUST NOT compare only displayed strings.


---

64. Truncation

A linker or runtime MUST NOT silently truncate a ULABI symbol identity.

If the underlying platform imposes a symbol-length limit, the implementation MUST use a collision-resistant encoding or an external symbol table.


---

65. Hash-Based Symbols

A platform MAY use a cryptographic digest of the canonical identity.

Conceptually:

CanonicalIdentity
       |
       v
Cryptographic Hash
       |
       v
Compact Symbol

If hashes are used, the algorithm and encoding MUST be explicitly identified.

A hash MUST NOT be treated as proof of semantic compatibility by itself.

The full contract metadata remains authoritative.


---

66. Hash Collision Handling

If a truncated or platform-specific hash is used, the implementation MUST retain sufficient metadata to detect collisions.

Silent collision acceptance is non-conforming.


---

67. Symbol Certificates

A future security profile MAY associate cryptographic signatures with symbol metadata.

For example:

Symbol Identity
+
Contract Metadata
+
Version
+
Publisher Identity
+
Signature

This is an extension of symbol authentication, not part of basic name mangling.


---

68. Supply-Chain Integration

Symbol identity MAY be incorporated into:

artifact signing;

dependency verification;

ABI manifests;

provenance metadata;

package verification.


However, name identity alone MUST NOT establish trust.

Trust is defined by the security and supply-chain specifications.


---

69. Deterministic Builds

Build systems SHOULD preserve deterministic ULABI symbol identities.

A rebuild from identical canonical declarations SHOULD produce identical canonical identities.

Implementation-specific addresses, timestamps, paths, or compiler-generated randomness MUST NOT affect canonical identity.


---

70. Debug Information

Debuggers SHOULD be able to map:

ULABI Identity
     |
     v
Source Declaration

and:

Source Declaration
     |
     v
ULABI Identity

Debug information MUST NOT change the normative identity.


---

71. Diagnostics

Diagnostic output SHOULD include:

ULABI identity
source name
namespace
entity kind
contract identity
version
profile
binding status

This allows developers to determine why two apparently identical symbols do not bind.


---

72. ABI Inspection

ULABI tooling SHOULD provide an inspection operation equivalent to:

inspect-symbol <identity>

which reports the canonical identity and relevant contract metadata.


---

73. ABI Difference Detection

Tooling SHOULD compare two symbol sets and classify differences as:

Added
Removed
Renamed source binding
Compatible contract change
Incompatible contract change
Version change
Profile change
Security requirement change

This should integrate with the compatibility specifications.


---

74. Source Renaming

Changing:

calculate

to:

compute

MUST NOT necessarily change the ULABI identity.

A source rename is not automatically an ABI break.

This is one of the primary reasons ULABI identity must remain separate from source names.


---

75. Namespace Renaming

Changing the namespace is potentially identity-breaking.

An implementation SHOULD treat:

org.example.math

and:

org.example.science

as distinct namespaces.

Migration SHOULD use explicit aliases or compatibility mappings.


---

76. Contract Renaming

A contract's human-readable name MAY change without changing its identity if the semantic contract remains the same.

The stable identifier is authoritative.


---

77. Semantic Changes

A source-compatible change that changes the ULABI contract MUST be evaluated by the compatibility system.

For example:

same source name
different ownership semantics

is not necessarily ABI-compatible.


---

78. Security-Relevant Changes

Changes to security-sensitive metadata MUST be considered explicitly.

Examples:

new capability requirement
new authorization requirement
new resource requirement
new execution privilege

A textual symbol match MUST NOT hide such changes.


---

79. Locality

A symbol identity MUST NOT silently change execution locality.

For example:

local function

must not silently become:

remote network call

merely because the same symbol is resolved remotely.

Locality semantics belong to the execution/distributed model.


---

80. Remote Symbols

Distributed implementations MAY resolve the same logical ULABI interface remotely.

However, remote invocation MUST explicitly preserve the additional semantics required by:

transport;

latency;

failure;

authentication;

authorization;

serialization;

distributed consistency.


The symbol identity alone does not make local and remote execution equivalent.


---

81. FFI Integration

The ULABI FFI layer MUST map:

Language Declaration
        |
        v
ULABI Symbol Identity
        |
        v
Calling Contract

The FFI MUST NOT invent a competing universal name-mangling scheme.

Language-specific FFI adapters MAY translate between native names and ULABI identities.


---

82. Object Model Integration

For object-oriented or object-like systems, symbol identity MUST remain separate from object layout.

A method identity can identify:

Type + Operation + Contract

without defining:

vtable
object header
inheritance representation
field layout

Those are specified elsewhere.


---

83. Type-System Integration

The name-mangling layer MUST reference canonical ULABI type identities.

It MUST NOT redefine:

primitive type semantics;

record layout;

enum semantics;

variant semantics;

generic semantics.


Those belong to the ULABI type-system specifications.


---

84. Calling Convention Integration

The symbol identity identifies what is being invoked.

The calling convention defines how it is invoked.

Therefore:

Symbol Identity
      |
      +----> Calling Convention
      |
      +----> Type Contract
      |
      +----> Memory Contract
      |
      +----> Error Contract

These concerns MUST remain separate.


---

85. Serialization Integration

Serialized representations MUST NOT be confused with linkage names.

A serialized type identifier MAY reference the same ULABI type identity.

However:

serialization encoding != symbol encoding

unless explicitly specified by a profile.


---

86. Compatibility Integration

Symbol identity resolution MUST integrate with:

docs/compatibility/backwards-compatibility.md
docs/compatibility/forwards-compatibility.md
docs/compatibility/feature-negotiation.md
docs/compatibility/capability-discovery.md
docs/compatibility/graceful-degradation.md

Name mangling identifies the candidate.

Compatibility determines whether the candidate can actually be used.


---

87. Capability Integration

A symbol MAY advertise required capabilities.

For example:

GPU
Filesystem
Network
SecureElement

The symbol identity MUST remain independent of the authorization decision.


---

88. Error Handling

Name-mangling operations MUST use the common ULABI error model.

At minimum, implementations SHOULD represent:

InvalidSymbol
InvalidEncoding
NamespaceNotFound
SymbolNotFound
AmbiguousSymbol
ContractMismatch
VersionMismatch
ProfileMismatch
UnauthorizedSymbol
SymbolCollision


---

89. Failure-Oriented Behaviour

Symbol resolution MUST fail closed.

If the identity cannot be verified, the implementation MUST NOT guess.

For example:

Expected:
    org.example.math.add

Found:
    org.example.math.add_v2

The loader MUST NOT silently substitute _v2 unless an explicit compatibility rule permits it.


---

90. Recovery

A failed symbol lookup MAY be recoverable through:

explicit alias;

compatible version;

supported profile fallback;

registered compatibility mapping.


Automatic substitution MUST remain policy-controlled.


---

91. No Silent Semantic Substitution

The following is non-conforming:

Requested function unavailable
        |
        v
Choose "similar" function automatically

ULABI requires deterministic and explicit resolution.


---

92. Canonical Identity Example

A conceptual symbol could be represented as:

namespace:
    org.example.math

kind:
    function

identifier:
    add

contract:
    <contract-id>

version:
    1

profiles:
    core

The canonical representation might then be encoded into a machine identifier.

The exact byte-level encoding belongs to the ULABI canonical identity/schema specification.


---

93. Example: Different Languages

Language A:

add

Language B:

plus

Language C:

Calculator.sum

All three can explicitly bind to:

namespace = org.example.math
operation = addition
contract = <same contract>
version = 1

Their local names remain independent.


---

94. Example: Overloads

add(Int32, Int32)
add(Float64, Float64)

MUST resolve to two distinct contract identities.

A lookup for:

add

without sufficient contract information MAY produce:

AmbiguousSymbol

rather than arbitrarily selecting one.


---

95. Example: Source Rename

Version 1:

calculate

Version 2 source:

compute

If the ULABI contract remains unchanged:

ULABI Identity = unchanged

The implementation simply changes its local binding.


---

96. Example: Breaking Change

Version 1:

read(Buffer) -> Result<Bytes, Error>

Version 2:

read(Buffer) -> Result<Stream, Error>

If the contract is incompatible, the implementation MUST expose a distinguishable identity/version.


---

97. Example: Architecture Migration

The same interface is implemented on:

x86-64
ARM64
RISC-V

The ULABI symbol identity SHOULD remain unchanged if the contract remains unchanged.

Architecture-specific calling conventions are selected independently.


---

98. Example: Multiple Linkers

The same ULABI identity may be encoded into:

ELF
Mach-O
PE/COFF
WebAssembly
runtime registry

The underlying platform symbols may differ.

The ULABI identity remains authoritative.


---

99. Canonical Identity Requirements

A conforming implementation MUST guarantee:

same canonical identity
        =>
same ULABI symbol identity

and:

different canonical identity
        =>
distinct identity

subject to the formally defined collision guarantees of the selected identity encoding.


---

100. Implementation Boundary

ULABI implementations SHOULD separate the following modules:

SourceNameAdapter
        |
        v
CanonicalIdentityBuilder
        |
        v
IdentityEncoder
        |
        v
SymbolRegistry
        |
        v
SymbolResolver
        |
        v
ContractVerifier

This separation prevents language-specific naming logic from contaminating the universal identity layer.


---

101. Required Invariants

The following invariants are mandatory.

INV-NAME-001

Canonical identity generation is deterministic.

INV-NAME-002

Source-language spelling does not determine semantic identity.

INV-NAME-003

Different entity kinds cannot silently collide.

INV-NAME-004

Incompatible contracts cannot silently share an identity.

INV-NAME-005

Symbol resolution cannot silently substitute an incompatible implementation.

INV-NAME-006

Platform-specific symbol encoding cannot alter canonical identity.

INV-NAME-007

Unicode normalization is deterministic.

INV-NAME-008

Aliases resolve to explicitly declared identities.

INV-NAME-009

Architecture migration does not inherently change identity.

INV-NAME-010

Security authorization is independent of symbol naming.


---

102. Security Requirements

A conforming implementation MUST:

validate canonical symbol encodings;

reject malformed identities;

prevent delimiter injection;

handle Unicode safely;

detect or safely represent confusable identifiers;

prevent unauthorized aliasing;

prevent ambiguous resolution;

prevent silent truncation;

validate contract metadata;

validate required profiles;

fail closed on unresolved identity.


Implementations SHOULD support:

signed symbol manifests;

provenance metadata;

namespace verification;

collision testing;

ABI-difference analysis.



---

103. Conformance Requirements

An implementation claiming conformance to ULABI Name Identity MUST pass tests covering:

1. deterministic identity generation;


2. namespace separation;


3. entity-kind separation;


4. overloaded operation separation;


5. generic identity;


6. method identity;


7. Unicode normalization;


8. confusable handling;


9. reserved-character escaping;


10. architecture independence;


11. source rename stability;


12. contract-change detection;


13. alias resolution;


14. ambiguity detection;


15. invalid identity rejection;


16. symbol truncation handling;


17. dynamic resolution;


18. static resolution;


19. compatibility integration;


20. security failure behavior.




---

104. Conformance Test Categories

Tests SHOULD be organized as:

tests/name_identity/
├── canonicalization/
├── namespaces/
├── functions/
├── methods/
├── overloads/
├── generics/
├── unicode/
├── escaping/
├── aliases/
├── versions/
├── profiles/
├── collisions/
├── resolution/
├── dynamic_loading/
├── security/
└── compatibility/


---

105. Reference Test Vectors

ULABI SHOULD maintain canonical test vectors containing:

Source Declaration
Canonical Semantic Record
Canonical Encoding
ULABI Identity
Expected Linkage Encoding
Expected Resolution Result

These vectors allow independent implementations to verify interoperability without sharing implementation code.


---

106. Reference Implementation Requirements

The reference implementation SHOULD expose:

parse_identity()
canonicalize_identity()
build_identity()
encode_identity()
decode_identity()
compare_identity()
resolve_symbol()
verify_contract()
verify_version()
verify_profile()

The reference implementation MUST be treated as an implementation of the specification, not as the specification itself.


---

107. Language Adapter Requirements

Each language adapter SHOULD provide:

source_name
source_type
source_module
source_namespace
source_overload
source_generic_metadata
source_visibility

and produce:

ULABI Symbol Identity

The adapter owns language-specific translation.

ULABI owns the resulting universal contract.


---

108. No Language-Specific Special Cases

The Core implementation MUST NOT contain branches such as:

if language == C++
if language == Rust
if language == Java
if language == Python
if language == Zamani
if language == Sankofa

Language-specific behavior belongs in adapters.

This preserves ULABI's neutrality.


---

109. Reference Architecture

The complete identity pipeline is:

Language A
                      |
                  Language B
                      |
                  Language C
                      |
                      v
              +----------------+
              | Language       |
              | Adapter        |
              +----------------+
                      |
                      v
              +----------------+
              | Semantic       |
              | Identity       |
              | Builder        |
              +----------------+
                      |
                      v
              +----------------+
              | Canonical       |
              | Encoder         |
              +----------------+
                      |
                      v
              +----------------+
              | ULABI Symbol   |
              | Identity       |
              +----------------+
                      |
             +--------+--------+
             |                 |
             v                 v
       Static Linker      Runtime Loader
             |                 |
             +--------+--------+
                      |
                      v
              Contract Verifier
                      |
                      v
                  Binding


---

110. Relationship to Other ULABI Documents

This document owns symbol identity and name translation.

It does not own:

Concern	Authoritative document

Core ABI	docs/abi/core-abi.md
Calling convention	docs/abi/calling-convention.md
Data types	docs/abi/data-types.md
Memory	docs/abi/memory-model.md
Language interoperability	docs/interoperability/language-interoperability.md
FFI	docs/interoperability/foreign-function-interface.md
Object model	docs/interoperability/object-model.md
Symbol resolution	docs/interoperability/symbol-resolution.md
Cross-language data	docs/interoperability/cross-language-data.md
Type compatibility	docs/type-system/type-compatibility.md
Versioning	ULABI-VERSIONING.md
Backward compatibility	docs/compatibility/backwards-compatibility.md
Forward compatibility	docs/compatibility/forwards-compatibility.md
Feature negotiation	docs/compatibility/feature-negotiation.md
Capability discovery	docs/compatibility/capability-discovery.md
Security	docs/security/security-model.md
Serialization	docs/distributed/serialization.md
Distributed calls	docs/distributed/remote-calls.md
Conformance	docs/standards/conformance.md
Test suite	docs/standards/test-suite.md


This separation prevents duplication.


---

111. Integration Contract

This document is designed so the surrounding documents can be completed without changing the fundamental name-mangling contract later.

The integration boundaries are fixed as follows:

Name Mangling
      |
      +--> Core ABI
      |
      +--> Type System
      |
      +--> FFI
      |
      +--> Symbol Resolution
      |
      +--> Versioning
      |
      +--> Compatibility
      |
      +--> Security
      |
      +--> Tooling
      |
      +--> Conformance

Each downstream component consumes the identity contract.

None should redefine it.


---

112. Required Implementation Modules

The implementation should be divided into independent modules.

Phase 1 — Independent Core Modules

These should be implemented first.

src/identity/mod.*

Core ULABI identity types.

Responsibilities:

identity representation;

entity kinds;

namespace;

contract identity;

version identity;

profile identity.



---

src/identity/canonical.*

Canonicalization engine.

Responsibilities:

canonical field ordering;

normalization;

deterministic representation;

semantic identity construction.



---

src/identity/encoding.*

Canonical identity encoding.

Responsibilities:

binary encoding;

textual encoding;

escaping;

decoding;

validation.



---

src/identity/unicode.*

Unicode security and normalization.

Responsibilities:

normalization;

invalid character detection;

control-character handling;

confusable diagnostics.



---

src/identity/hash.*

Optional compact identity encoding.

Responsibilities:

cryptographic hashing;

digest encoding;

collision verification;

algorithm identification.



---

src/identity/version.*

Contract/version identity.

Responsibilities:

interface versions;

ABI profile versions;

compatibility identity.


This module consumes the version model rather than redefining global ULABI versioning.


---

Phase 2 — Symbol Modules

After the independent identity layer:

src/symbol/mod.*

Symbol abstraction.

Responsibilities:

exported symbols;

imported symbols;

visibility;

aliases;

symbol metadata.



---

src/symbol/registry.*

Symbol registry.

Responsibilities:

symbol registration;

lookup;

namespace indexing;

alias indexing.



---

src/symbol/resolver.*

Symbol resolution.

Responsibilities:

exact identity lookup;

ambiguity detection;

compatibility-aware lookup;

failure reporting.



---

src/symbol/alias.*

Explicit alias handling.

Responsibilities:

alias registration;

alias validation;

alias resolution;

alias security.



---

Phase 3 — Contract Integration

src/contract/mod.*

Contract metadata abstraction.

Responsibilities:

contract identity;

parameter references;

return references;

effects;

execution semantics;

capabilities.



---

src/contract/verifier.*

Contract verification.

Responsibilities:

identity verification;

type compatibility;

version compatibility;

profile compatibility.



---

Phase 4 — Language Integration

src/adapters/mod.*

Common adapter interface.


---

src/adapters/language.*

Language-neutral adapter contract.


---

src/adapters/<language>/*

Language-specific adapters.

Examples:

src/adapters/c/
src/adapters/cpp/
src/adapters/rust/
src/adapters/go/
src/adapters/java/
src/adapters/python/
src/adapters/swift/
src/adapters/kotlin/
src/adapters/fortran/
src/adapters/ada/

These are optional implementations, not part of the ULABI Core specification.

Zamani and Sankofa may independently implement adapters, but neither belongs inside ULABI Core.


---

113. Linker Integration Modules

After symbol identity and resolution:

src/linker/symbols.*
src/linker/identity_resolver.*
src/linker/abi_validator.*

Responsibilities:

resolve ULABI identities;

preserve identity metadata;

detect incompatible symbols;

generate platform linkage names.



---

114. Loader Integration Modules

src/loader/symbol_loader.*
src/loader/identity_validator.*
src/loader/contract_validator.*

Responsibilities:

runtime symbol discovery;

identity verification;

contract verification;

profile verification;

secure binding.



---

115. FFI Integration Modules

src/ffi/mod.*
src/ffi/imports.*
src/ffi/exports.*
src/ffi/bindings.*
src/ffi/signatures.*

Responsibilities:

Native Declaration
       |
       v
ULABI Identity
       |
       v
ULABI Contract


---

116. Tooling Modules

src/tools/symbol_inspector.*
src/tools/abi_diff.*
src/tools/symbol_validator.*
src/tools/symbol_dump.*

Responsibilities:

inspect symbols;

compare ABI identities;

validate manifests;

generate diagnostics.



---

117. Schema Files

The following schemas should eventually be added:

schemas/
├── symbol-identity.schema.json
├── namespace.schema.json
├── contract-identity.schema.json
├── symbol-manifest.schema.json
├── alias.schema.json
├── profile-identity.schema.json
└── symbol-resolution.schema.json

The exact schema language may evolve, but the semantic structures defined by this document must remain stable.


---

118. Example Files

Recommended examples:

examples/name-mangling/
├── basic-function/
├── overloaded-function/
├── generic-function/
├── method/
├── source-rename/
├── version-change/
├── unicode/
├── alias/
├── architecture-port/
└── cross-language/

Each example should show:

Source Declaration
        ↓
ULABI Binding
        ↓
Canonical Identity
        ↓
Linkage Representation
        ↓
Resolution


---

119. Conformance Files

Recommended:

conformance/name-mangling/
├── identity/
├── canonicalization/
├── unicode/
├── overloads/
├── generics/
├── aliases/
├── versions/
├── resolution/
├── collisions/
├── security/
└── compatibility/


---

120. Required Integration Order

To satisfy the requirement that each file can be completed without reopening it because another document was later changed, implementation should proceed in this order.

Stage 1 — Foundation

1. schemas/symbol-identity.schema.json


2. src/identity/mod.*


3. src/identity/canonical.*


4. src/identity/unicode.*


5. src/identity/encoding.*


6. src/identity/hash.*



These establish the identity substrate.


---

Stage 2 — Symbols

7. src/symbol/mod.*


8. src/symbol/registry.*


9. src/symbol/alias.*


10. src/symbol/resolver.*



These consume the identity substrate.


---

Stage 3 — Contracts

11. src/contract/mod.*


12. src/contract/verifier.*



These connect symbol identity to actual ULABI contracts.


---

Stage 4 — Language Adapters

13. src/adapters/mod.*


14. src/adapters/language.*


15. individual language adapters



These translate source declarations into ULABI identities.


---

Stage 5 — Linker / Loader

16. src/linker/symbols.*


17. src/linker/identity_resolver.*


18. src/linker/abi_validator.*


19. src/loader/symbol_loader.*


20. src/loader/identity_validator.*


21. src/loader/contract_validator.*




---

Stage 6 — FFI

22. src/ffi/mod.*


23. src/ffi/imports.*


24. src/ffi/exports.*


25. src/ffi/bindings.*


26. src/ffi/signatures.*




---

Stage 7 — Tooling

27. src/tools/symbol_inspector.*


28. src/tools/symbol_validator.*


29. src/tools/symbol_dump.*


30. src/tools/abi_diff.*




---

Stage 8 — Conformance

31. identity tests


32. canonicalization tests


33. Unicode tests


34. overload tests


35. generic tests


36. alias tests


37. version tests


38. resolution tests


39. security tests


40. compatibility tests




---

121. Files That Must Not Be Made Responsible for Name Mangling

To prevent architectural repetition, the following documents/modules should not redefine name identity:

docs/abi/calling-convention.md
docs/abi/memory-model.md
docs/abi/data-types.md
docs/runtime/runtime-interface.md
docs/security/security-model.md
docs/distributed/serialization.md
docs/distributed/remote-calls.md

They consume ULABI identity where necessary.


---

122. Final Architectural Rule

ULABI must maintain this separation:

Language Source Name
        |
        | language-specific
        v
Language Adapter
        |
        | ULABI-defined
        v
Canonical ULABI Identity
        |
        +---- Contract
        +---- Version
        +---- Profiles
        |
        v
Platform Linkage Encoding

Never:

C++ mangling -> ULABI
Rust mangling -> ULABI
Java mangling -> ULABI
Zamani mangling -> ULABI
Sankofa mangling -> ULABI

Instead:

C++ --------\
Rust --------\
Java ---------\
Python --------> ULABI Identity <-------- Any Future Language
Zamani -------/
Sankofa -----/

ULABI therefore remains what ULABI-DESIGN.md establishes: the universal interoperability contract, not a programming language's naming convention or a compiler's private ABI mechanism.

This also fits the repository's existing separation of language interoperability, FFI, types, calling convention, compatibility, and distributed interfaces rather than duplicating those responsibilities.