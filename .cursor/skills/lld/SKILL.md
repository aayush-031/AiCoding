---
name: lld
description: >-
  Guides phased Low-Level Design for concurrent Java components. Produces
  architecture sketches (class diagrams, API contracts, concurrency design)
  before tests or implementation. Use when the user invokes /lld, /lld-design,
  or asks for LLD or structural design.
disable-model-invocation: true
---

# LLD — Low-Level Design Workflow

## Trigger

Apply this skill when the user invokes **`/lld`** or **`/lld-design`**, or asks to sketch architecture before coding.

If the user has not named a target component, ask what they want to design before producing Phase 1 output.

## Core Constraint

**Never generate a fully implemented solution all at once.** Follow the strict 3-phase lifecycle and stop at the end of the current phase. Wait for explicit user approval before advancing.

```
Phase 1: Planning & Architecture  → STOP, wait for approval
Phase 2: Test Cases (TDD)         → STOP, wait for approval
Phase 3: Incremental Implementation + terminal verification
```

---

## Phase 1 Output Template

When `/lld` is invoked, **halt all code production** and deliver a Markdown design containing:

1. **Class diagram** — ASCII or Mermaid showing inheritance, composition, and dependency relationships.
2. **Class & interface breakdown** — Each type with fields, methods, and technical responsibilities.
3. **Concurrency Design** — Which objects are mutable, lock domains, invariants protected against race conditions, and why chosen primitives (`ReentrantReadWriteLock`, `LongAdder`, etc.) fit.

Do **not** write test classes or full business-logic implementations in Phase 1 unless the user explicitly requests a later phase.

### Phase 1 sketch guidelines

- Target **Java 21+** (records for DTOs where appropriate).
- Defer method bodies, test suites, and runnable implementation to later phases unless the user explicitly requests them.
- Document data structures with complexity rationale for core read/write/lookup paths.
- Specify concurrency primitives (`ReentrantReadWriteLock`, `LongAdder`, etc.) and which mutable state each protects.
- Use `LongAdder` or `LongAccumulator` for high-contention metrics counters.

### Example Phase 1 deliverable shape

```markdown
## Class Diagram
[Mermaid or ASCII]

## Public API
- method signatures + javadoc-level contract

## Internal Types
- fields, mutability, responsibilities

## Concurrency Design
- mutable state inventory
- lock domains and acquisition order (deadlock prevention)
- visibility invariants
```

---

## Phase 2 — Test Cases (TDD)

Only after Phase 1 approval. Write unit + integration tests covering:

- Boundary thresholds relevant to the component
- Heavy contention (`CountDownLatch` + `ExecutorService`, dozens of threads)
- Erroneous or invalid inputs
- Metric counter correctness under stress (if applicable)
- Ordering and state invariants under out-of-order execution

Do **not** implement core business logic yet — tests may not compile until Phase 3.

---

## Phase 3 — Incremental Implementation

Only after Phase 2 approval. Implement in small, verifiable slices aligned with the approved design. After each slice, run terminal verification (`javac`, `java`, or project test runner) and report results.

---

## Architecture Standards (all phases)

- **SOLID** — decouple via interfaces; one responsibility per class.
- **Fail-fast** — reject invalid params/states immediately with clear exceptions.
- **No broad `synchronized` methods** — fine-grained locks only.
- **No nested lock paths that can deadlock** — document and enforce lock ordering.
- **Optimal complexity** for core read/write/lookup paths — no unnecessary collection scans.

---

## Anti-Patterns

- Jumping straight to a complete implementation in Phase 1
- Implementing core business logic when asked to sketch only
- Skipping the Concurrency Design section
- Proceeding to Phase 2/3 without user approval
