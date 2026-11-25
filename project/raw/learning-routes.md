# Raw input: learning routes

| Field | Value |
| --- | --- |
| Status | Raw future-work input / non-authoritative; to be written after the main specification is done |
| Audience | Future documentation work on learning paths for people and agents |
| Applies To | Step-by-step primers, "if you know X" language bridges, and the evidence for writing them |
| Owns | The maintainer's definitions of the two routes, their costs and limits, the beginner-corpus test, and the open question about owner repetition |
| Does Not Own | Documentation architecture ([documentation](../documentation.md)); the comparative guide's content ([similarity](similarity.md)); current language owners |
| Source / Provenance | Work item `040` (element and contained stances), discussion of teachability and AI readers |

## Why

An AI trained heavily on another language already knows that language's
prerequisites. For Zax, a reader, human or AI, knows only what it has read.
Today the current owners repeat prerequisites across documents so that a
reader entering anywhere can follow. That works, but it is repetitive, and a
missed prerequisite still causes mistakes. A compiler with good diagnostics
is the long-term fix for both people and AI; these routes are the
documentation part.

## Two routes

- **A primer** is a step-by-step learning guide. Each step assumes the
  previous steps were read, so it can be short and unrepetitive. It links to
  the owners for complete rules instead of redefining them. It is costly to
  keep correct, because one language change can affect every later step, so it
  is written after the core specification settles.
- **A language bridge** ("if you know X, here is how Zax does it") may be
  entered anywhere. It explains Zax in terms the reader already knows, then
  links into the owners. Each bridge claim names where the analogy stops,
  because a near-right analogy misleads: "stances are like Rust ownership" is
  right enough to be dangerous. The comparative content itself is preserved in
  [similarity](similarity.md).

## The beginner-corpus test

Write a small set of ordinary programs with defaults only, and record which
concepts each one forces on the writer: reading a file, parsing lines into
structs, sorting and filtering a collection, a type with methods, and a
callback. The result shows where the common path is heavier than intended,
for example that a slice parameter needs `loan` or `copy`.

## Open question

May owners assume primer knowledge and repeat less? That would change the
direct-entry rule in [documentation](../documentation.md), which assumes a
reader may enter any owner or section directly, and needs its own discussion.
The current lean is no, or only for a short, explicitly named baseline,
because owners are also what a specification would later rest on.

## Activation and retirement

Activate when the main specification is done and documentation clean-up
begins. Move accepted routes into a reader route the documentation
architecture defines, then retire this file and remove it from
[the raw index](README.md).
