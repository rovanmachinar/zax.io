# Raw input: safety boundaries

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining safety guarantees and unsafe boundaries |
| Applies To | Safe-subset guarantees, unsafe mechanisms, and comparative safety input |
| Owns | Preservation of aligned pressures and unresolved comparison material |
| Does Not Own | Accepted safety guarantees or unsafe syntax |
| Source / Provenance | Work item `001`, Zax purpose and design principles |

## Aligned direction

Maximizing compiler-enforced safety is not Zax's organizing goal. Zax should
nevertheless define a safe subset with explicit guarantees.

Code using only safe operations and interfaces should receive the guarantees
assigned to that subset. Operations leaving those guarantees must be
recognizable and auditable without forcing the whole language into one
restrictive ownership model.

The final mechanism need not use an `unsafe { ... }` scope. Narrow operation
markers, unsafe declarations or interfaces, effect propagation, and reviewed
safe wrappers around unsafe implementations require evaluation.

## Guarantee categories to investigate

- Type integrity
- Initialization
- Bounds
- Pointer validity
- Lifetime and use-after-free
- Aliasing
- Data-race boundaries
- Arithmetic behavior
- Panic behavior
- FFI and assembly trust
- Resource exhaustion
- Deadlock and starvation
- Logic and broader security correctness

The last categories may be explicit non-guarantees rather than properties the
compiler can prove.

## Comparative input

Rust is a comparison point, not an opponent.

Preserve investigation of:

- the value of safe Rust's memory-safety and data-race guarantees;
- borrow-checker restrictions and ownership-oriented redesign;
- `Arc`, `Mutex`, `RefCell`, state machines, and indirection as both real costs
  and possible cognitive overhead;
- unsafe implementations in standard libraries and dependency chains;
- FFI invariants;
- deadlock, starvation, panic, resource, logic, cryptographic, authorization,
  and business-rule non-guarantees;
- C++ mitigation through RAII, ownership conventions, smart pointers,
  sanitizers, static analysis, review, and tooling;
- AI-assisted analysis as broad but probabilistic support rather than a
  deterministic guarantee; and
- CHERI-style pointer capabilities and their limits around temporal lifetime,
  concurrency, resources, and logic.

Future work must define Zax's promises first, then compare them carefully.

## Activation and retirement

Activate this input before publishing safety guarantees or defining unsafe
boundaries. Consume its findings through that work and retire or archive this
placeholder afterward.
