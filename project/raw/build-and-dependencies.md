# Raw input: build execution and source dependencies

| Field | Value |
| --- | --- |
| Status | Raw placeholder / non-authoritative |
| Audience | A future numbered work item defining build-time execution or dependency behavior |
| Applies To | Trust, sandboxing, source resolution, immutable identity, provenance, and durable builds |
| Owns | Preservation of aligned pressures, threats, and unresolved mechanisms |
| Does Not Own | Accepted dependency schemas, security policy, or implementation contracts |
| Source / Provenance | Work item `001`, Zax purpose and design principles |

## Aligned pressure

Zax integrates build-time execution and source dependency resolution into the
language build. It does not require an external package-management step or an
official package store, though optional catalogs or package schemas may exist.

Build-time code is executable code. It is trusted unless the build environment
deliberately constrains it.

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
dependency declaration and locking, sandbox policy, or an implementation
contract. Consume its findings through that work and retire or archive this
placeholder afterward.
