# ULABI Design Document

**Universal Language Application Binary Interface (ULABI)**

**Status:** Draft / Design Phase  
**Version:** 0.1.0-draft

---

## 1. Vision

ULABI is an open, language-neutral interoperability standard intended to allow independently designed programming languages, runtimes, libraries, and systems to exchange values, invoke functions, manage resources, and communicate execution semantics through stable binary contracts.

ULABI is not a programming language, compiler, operating system, or mandatory runtime.

It is an interoperability specification with independent implementations.

---

## 2. Core Principles

### 2.1 Language Neutrality

ULABI must not be designed around any single programming language.

### 2.2 Implementation Independence

Implementations may use different:

- programming languages;
- compilers;
- runtimes;
- memory models;
- operating systems;
- architectures;
- internal representations.

### 2.3 Explicit Contracts

Interfaces must explicitly define:

- types;
- ownership;
- resource behavior;
- errors;
- compatibility requirements;
- execution semantics where applicable.

### 2.4 Safety by Design

Malformed or hostile input must be rejectable without:

- undefined behavior;
- memory corruption;
- uncontrolled allocation;
- uncontrolled recursion;
- uncontrolled resource consumption.

### 2.5 Deterministic Encoding

Canonical representations should allow independent implementations to produce and consume compatible binary representations.

### 2.6 Versionability

Interfaces must support explicit versioning and controlled evolution.

### 2.7 No Vendor Lock-In

No single:

- company;
- programming language;
- runtime;
- compiler;
- operating system;
- hardware platform

should be required to implement ULABI.

### 2.8 Open Development

The specification, reference implementations, tools, and conformance tests should be developed transparently.

### 2.9 Interoperability Before Convenience

Language-specific implementation details should not unnecessarily leak across the interoperability boundary.

### 2.10 Independent Language Ecosystems

ULABI support in Zamani and Sankofa, if introduced in the future, must be implemented independently.

ULABI must not make Zamani dependent on Sankofa or Sankofa dependent on Zamani.

---

# 3. Scope

The initial ULABI scope includes:

- primitive values;
- strings;
- byte sequences;
- records;
- variants;
- enums;
- lists;
- optional values;
- result values;
- function calls;
- errors;
- resource handles;
- ownership;
- lifetime semantics;
- canonical binary encoding;
- interface versioning;
- compatibility;
- conformance testing.

Future specifications may cover:

- asynchronous functions;
- futures;
- streams;
- cancellation;
- capability-based security;
- identity;
- authorization;
- debugging;
- source mapping;
- tracing;
- profiling;
- zero-copy interoperability;
- shared memory;
- distributed interoperability.

---

# 4. Non-Goals

ULABI will not initially attempt to:

- replace every existing machine ABI;
- replace platform C ABIs;
- mandate a universal memory-management model;
- mandate garbage collection;
- mandate ownership/borrowing;
- mandate manual memory management;
- require a universal runtime;
- define a universal programming language;
- require every implementation to use Rust;
- require WebAssembly;
- merge Zamani and Sankofa;
- make Zamani the foundation of Sankofa;
- make Sankofa the foundation of Zamani.

---

# 5. Architectural Model

ULABI should be designed as a layered interoperability system.

```text
                    ULABI
                      |
        +-------------+-------------+
        |             |             |
     Core ABI      Type ABI     Resource ABI
        |             |             |
        +-------------+-------------+
                      |
              +-------+-------+
              |               |
          Error ABI      Compatibility ABI
              |               |
              +-------+-------+
                      |
              Concurrency ABI
                      |
                 Security ABI