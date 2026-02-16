# RFC: Structural Typing — Equivalence Rules

- RFC ID: 2026-02-16-structural-typing-equivalence
- Status: Draft
- Author(s): zax-planner (editorial draft)
- Created: 2026-02-16
- Related: (none yet)

## Summary
Define when two record/struct types are considered *structurally equivalent* in Zax. Equivalence is based on a type’s *field set* (field **names + field types**), independent of declaration order. Layout/ABI order is treated as a separate concern that can be specified explicitly when required.

## Goals
- Make “structural typing” precise and testable.
- Ensure field **names matter**: renaming a field breaks equivalence.
- Ensure field **order does not matter** for equivalence.
- Leave room for explicit layout/ABI control without leaking it into equivalence.
- Specify how equivalence interacts with:
  - extra/missing fields (width subtyping vs exact match),
  - nested records,
  - generics,
  - reflection (“same shape”).

## Non-goals
- Fully specifying the memory layout / ABI rules (only the separation-of-concerns boundary).
- Defining nominal typing or inheritance.
- Solving all variance/subtyping questions for function types (out of scope unless needed to define “field types equal”).

---

## Motivation
Zax’s structural typing is a core design goal, but without a clear equivalence relation it becomes difficult to reason about assignments, generics instantiation, reflection, and stable compilation. We want a simple mental model:

- “Same fields (by name) with the same types” → same record shape.
- How it is *laid out* in memory is a separate opt-in choice.

---

## Proposal

### 1) Definitions

**Record / struct type (record-like aggregate)**: a type consisting of named fields (e.g., `{ x: i32, y: i32 }`).

**Shape**: the mapping from field name → field type (and potentially field attributes; see Open Questions).

**Structural equivalence** (`≡`): two record types are equivalent iff they have the same shape under the rules below.

### 2) Core equivalence rule (records)

Two record types `R1` and `R2` are structurally equivalent, written `R1 ≡ R2`, if and only if:

1. They have the **same set of field names**.
2. For each field name `f`, the field types are equivalent by the language’s type equivalence (`typeEq`) relation:  
   `typeEq(R1.f, R2.f)` holds for all fields `f`.
3. **Field order is ignored** for equivalence.

This directly satisfies the provided examples:

**Equivalent**
- `A { x: i32, y: i32 } ≡ B { y: i32, x: i32 }`

**Not equivalent**
- `C { x: i32, y: i32 } ≢ D { x: i32, y: u32 }` because `typeEq(i32, u32)` is false.

### 3) Field names matter
Field identity is determined by **name**. Therefore:
- `{ x: i32, y: i32 } ≢ { a: i32, b: i32 }` (even if “positionally” similar)
- Renaming a field is a breaking change for equivalence.

### 4) Order-independent equivalence vs explicit layout/ABI

Equivalence ignores order, but layout may need order. The language should allow a *separate* mechanism to specify layout/ABI order (and possibly packing/alignment) without affecting structural equivalence.

**Normative rule**:
- Layout annotations/controls **do not participate** in structural equivalence, unless explicitly specified otherwise (see Open Questions).

Concretely:
- Two records can be structurally equivalent but have different chosen layouts.
- Assignability/conversion may require a separate “layout-compatible” check when operating at raw-memory boundaries (FFI, transmute-like operations, etc.).

*(This RFC defines equivalence; it does not define layout compatibility.)*

### 5) Extra/missing fields: exact-shape vs width subtyping

We need to pick one of these (or support both with explicit operators). This RFC proposes a default and allows an explicit escape hatch.

#### Option A — Exact-shape only (recommended default for Zax v1)
Records are equivalent/assignable only when they have exactly the same field set.

- `Rsmall = { x: i32, y: i32 }`
- `Rbig   = { x: i32, y: i32, z: i32 }`

Then:
- `Rsmall ≢ Rbig`
- No implicit assignment between them purely via structural typing.

**Rationale**: keeps compilation stability, avoids surprising “hidden” field drops, and plays well with low-level memory use.

#### Option B — Width subtyping (opt-in, explicit)
Allow `Rbig` to be usable where `Rsmall` is expected *only* via an explicit operation, such as:
- a projection/cast: `as {x:i32,y:i32}`
- or a pattern/projection expression producing a new value.

If Zax wants width subtyping *implicitly*, specify:
- `Rbig <: Rsmall` if `Rbig` contains all fields of `Rsmall` with equivalent types.
- But **equivalence** remains exact (equivalence is not subtyping).

**This RFC’s stance**:
- Define **equivalence as exact-shape**.
- If width subtyping is desired, introduce it as a **separate relation** (`<:`) and decide implicitness separately.

### 6) Nested records
Equivalence is **recursive**: if a field type is itself a record, equivalence uses the same rules.

Example:
- `{ p: { x:i32, y:i32 } } ≡ { p: { y:i32, x:i32 } }`

As long as the nested record shapes match (by name+type, ignoring order).

### 7) Generics / parametric records

#### 7.1 Generic record declarations
A generic record definition introduces a type function, e.g.:
- `Pair<T,U> = { fst: T, snd: U }`

Equivalence is determined on **instantiated** types:
- `Pair<i32, i32> ≡ { fst:i32, snd:i32 }`
- `Pair<i32, u32> ≢ Pair<i32, i32>`

#### 7.2 Type parameters and constraints
If Zax has constrained generics (e.g., `T: SomeTrait`), equivalence should use the instantiated type arguments, not the constraints.

Open point: whether two uninstantiated generic types can be “equivalent as templates” is usually not needed; keep it simple:
- Only fully-formed types participate in `≡`.

### 8) Reflection: reporting “same shape”
Reflection should expose the *shape* in a canonical, order-independent way.

Proposal:
- `reflect.shape(T)` returns a normalized description:
  - list of `(fieldName, fieldTypeId)` sorted by `fieldName` (or stable hash order),
  - plus any shape-relevant attributes if adopted.
- `reflect.sameShape(T, U)` returns true iff `T ≡ U`.

Important:
- If layout is separate, reflection should offer separate queries:
  - `reflect.layout(T)` (explicit order, offsets, alignment, packing)
  - `reflect.sameLayout(T, U)` (if needed)

---

## Alternatives considered
1. **Field order matters** (positional records)  
   Rejected: conflicts with the stated constraint and reduces flexibility for refactors.

2. **Names don’t matter** (pure positional typing)  
   Rejected: violates constraint and harms readability/safety.

3. **Implicit width subtyping by default**  
   Risky for low-level/data-oriented code; can introduce accidental field loss and subtle bugs. Better as explicit projection or separate subtype relation.

---

## Backwards compatibility / stability
- Decl-order changes do not affect equivalence (good for refactors).
- Renaming a field is a breaking change (explicitly intended).
- Changing a field type is breaking (expected).
- Adding/removing fields changes shape, so it breaks equivalence; if width subtyping is later added, it must be designed to avoid undermining compilation stability guarantees.

---

## Open questions (to resolve in follow-up RFCs or an appendix)
1. **Width subtyping**: exact-shape only vs explicit projection vs implicit subtyping (`<:`). What’s the intended ergonomics?
2. **Shape-relevant attributes**: do `mut`, `const`, visibility, alignment, padding, or “noalias” qualifiers participate in equivalence?
3. **Duplicate field names**: presumably illegal; confirm.
4. **Type equivalence for field types**: is it nominal for some types (e.g., distinct typedef/newtype) or purely structural everywhere?
5. **Recursive types**: how to handle self-referential record types in equivalence + reflection without infinite recursion (hash-consing / IDs).
6. **Layout boundary**: what operations require “layout-compatible” vs just “shape-equivalent”?
