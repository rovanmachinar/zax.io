# Raw input: Nothing instances

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative |
| Audience | Future work defining `Nothing` resident instances, pointer emptiness, monitored access, and custom fallback behavior |
| Applies To | Real global `Nothing` instances, pointer presence, generated member-access checks, custom `Nothing`, compatible overlap, and thread safety |
| Owns | Preservation of the current candidate model, representative behavior, deciding questions, activation pressure, and retirement criteria |
| Does Not Own | Accepted pointer presence or ownership behavior |
| Source / Provenance | Legacy `nothing.md`; work item `014` follow-up |

## Candidate mental model

A pointer that semantically points to `Nothing` may point to a real global
resident instance rather than a null address.

`?pointer` still reports false because the pointer denotes `Nothing`, regardless
of physical representation.

The default language-provided `Nothing` instance and a programmer-provided
custom `Nothing` have different proposed behavior.

## Default monitored Nothing

Candidate behavior:

- one real global resident instance supplies the type's default `Nothing`;
- accessing its member storage may trash shared sentinel state;
- compiler-generated monitoring may panic before a member read or write;
- a type-defined final function needs a generated `_` check only when execution
  may access `_` members;
- a member-independent function, including an appropriate `once`-like operation,
  need not pay the check;
- compiler modes may enable or disable monitoring; and
- disabling a required check under an unsafe guarantee makes an actual invalid
  access undefined.

Illustrative source shape only:

```zax
MyValue :: type {
  member : Integer

  inspect final : ()() = {
    print(_.member) // may require generated Nothing protection
  }

  identity final : (result : Integer)() = {
    return 0 // no member access; no Nothing-member check required
  }
}
```

This example does not establish final `Nothing` declaration syntax.

## Custom Nothing

A programmer may provide a legitimate global instance as a type's custom
`Nothing`.

Candidate behavior:

- it receives ordinary type-defined function calls;
- its functions may return neutral behavior, perform useful work, or panic;
- it does not receive default monitored-member checks unless its own contract
  requests them;
- the programmer owns mutation and synchronization behavior; and
- being global provides no intrinsic thread safety.

A custom `Nothing` remains semantically `Nothing` for pointer presence even when
its functions deliberately accept calls.

## Compatible overlapping storage

Future work may permit several types to share compatible `Nothing` backing
storage. It must define:

- representation compatibility;
- size and alignment;
- active resident-instance meaning;
- aliasing and mutation;
- thread safety;
- whether one type may observe another type's trashed sentinel state;
- and how compiler monitoring identifies the correct logical type.

Physical overlap cannot by itself establish simultaneous typed lifetimes.

## Representation and exposure

Future work must decide:

- whether `Nothing` uses a dedicated global sentinel, compatible shared sentinel
  storage, a tagged pointer state, a null-like address with synthesized
  behavior, or another representation;
- how raw and managed pointers differ in storing, detecting, and dereferencing
  `Nothing`;
- whether ownership control blocks refer to the sentinel or represent
  `Nothing` without one;
- how pointer equality and presence observe the representation;
- how source reflection reports custom versus default `Nothing`;
- which representation facts are part of ABI and interoperability contracts;
- and whether representation may vary by target without changing source
  behavior.

The candidate real-global-instance model remains the preferred conceptual input,
but this file does not yet establish one required representation.

## Pointer and optional consequences

Future work must preserve:

- pointer `Nothing` remains distinct from optional absence;
- a present optional pointer may itself contain `Nothing`;
- raw non-`Nothing` state proves no lifetime or provenance;
- weak termination remains distinct from a live pointer to `Nothing`;
- pointer ownership never keeps a shared default sentinel destructible; and
- custom `Nothing` behavior does not make arbitrary missing allocations present.

Current pointer presence and ownership are defined by
[Zax pointers and arenas](../../language/pointers-and-arenas.md).

## Panic and unsafe controls

Future work must distinguish:

- ordinary monitored access that panics;
- compiler modes that add diagnostic checks;
- explicit unchecked access that omits a check;
- a programmer guarantee that the access cannot reach `Nothing`;
- actual sentinel corruption;
- and custom `Nothing` behavior that is defined rather than erroneous.

Unchecked access is safety responsibility, not intent acknowledgement. A false
guarantee may have undefined consequences.

## Activation and retirement

Activate this input when `Nothing` construction, pointer representation,
pointer dereference, custom fallback instances, generated `_` checks, compatible
sentinel storage, or `Nothing` thread safety is reviewed.

Move accepted behavior into type, pointer, lifetime, construction, safety,
concurrency, and representation owners, then retire this file.
