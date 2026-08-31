# Raw input: build execution and source dependencies

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining build-time execution or dependency behavior |
| Applies To | Trust, sandboxing, source resolution, immutable identity, provenance, durable builds, and required language support supplied by the toolchain |
| Owns | Preservation of aligned pressures, threats, required-support and linkage questions, and unresolved mechanisms |
| Does Not Own | Accepted build/dependency semantics or implementation contracts |
| Source / Provenance | Work item `001`, Zax purpose and design principles; operator and foundational-support pressure |

## Aligned pressure

Zax integrates build-time execution and source dependency resolution into the
language build. It does not require an external package-management step or an
official package store, though optional catalogs or package schemas may exist.

Build-time code is executable code. It is trusted unless the build environment
deliberately constrains it.

## Required language support and toolchain completeness

Zax does not require one mandatory conventional system library, but an
implementation cannot omit the support the language itself requires. Preserve the
distinction between:

- optional library APIs such as collections, I/O, networking, and frameworks; and
- required language support for fundamental types, protected primitive
  operations, and the runtime or lifetime behavior that source uses.

Protected intrinsic operator families are current concrete examples. Exact
`Boolean && Boolean` and `Boolean || Boolean` are protected short-circuit
operations; protected integer arithmetic, policy, bitwise, shift, reduction,
count, magnitude, mutation, and swap forms likewise belong to required language
support where the current
[operator catalog](../../language/operator-catalog.md) makes them available.
Their shared programmer model is owned by
[Zax operators](../../language/operators.md), and the foundational promise by
the [Zax language vision](../../language/vision.md).

A conforming implementation may satisfy a required operation through instructions,
constant folding, emitted target code, an intrinsic, or an automatically linked
helper. Those are implementation choices, and no lowering is promised.

```zax
// error: the protected Boolean || Boolean operation cannot be replaced
operator binary '||' final :
  (result : Boolean)(lhs : Boolean, rhs : Boolean) = {
  return asmCompare(lhs, rhs)
}
```

Future build and toolchain work must decide:

- how automatically linked helper objects or runtime fragments are located,
  versioned, and pinned;
- how required-support availability interacts with cross-compilation, freestanding
  targets, and host/target separation;
- how a toolchain reports missing required support, given that this is a broken
  or incomplete toolchain rather than the diagnostic the source was meant to
  produce; and
- how required support interacts with durable, reproducible, and locked builds.

Exact compiler and runtime architecture remains undecided here.

## Sandboxing approaches

Two complementary layers need future design:

1. Restricted build-time capabilities, libraries, compiler services, and host
   access.
2. Externally isolated builds using disposable machines, VMs, containers, or
   hosted workers without trusted credentials or privileged resources.

The complete execution surface includes filesystem, network, processes,
environment variables, dynamic loading, FFI, compiler intrinsics, and transitive
build code.

## Immutable dependency identity

Branches, tags, and version names are human-friendly selectors rather than
immutable identities.

A future dependency model should consider recording:

- repository identity;
- requested selector;
- resolved commit identity;
- canonical source-tree or content digest;
- recursive dependency identities;
- signatures or provenance; and
- build-time capabilities required.

Existing locked builds should not silently accept a moved tag. An explicit
update can report and accept newly resolved content. Development may permit
floating branches while durable or release builds require locked identities.

## Threat input

Preserve consideration of:

- moved, deleted, or recreated tags;
- compromised repository accounts;
- deletion and namespace takeover;
- dependency confusion;
- transitive mutation;
- submodules, LFS, generated content, and external downloads;
- malicious build-time code;
- cache poisoning;
- unsigned or unverifiable history;
- redirects and repository identity changes;
- hash migration or collision concerns;
- host-dependent behavior;
- deterministic and hermetic build policies;
- cross-compilation and host/target separation; and
- caching and incremental invalidation.

## Activation and retirement

Activate this input before defining the lasting build/dependency owner,
dependency declaration and locking, sandbox policy, required-support and helper
linkage behavior, or an implementation contract. Consume its findings through
that work and retire or archive this placeholder afterward.
