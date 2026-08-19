# Zax project guidance

This directory contains the public-safe project guidance and working material
used to refine Zax. It is not part of the ordinary language-documentation
reading path.

## Start here

| Task | Action |
| --- | --- |
| Start a new Zax design session | The language maintainer copies the block from [the handoff source](handoff.md) into the new session |
| Recover the same session after context compaction | The language maintainer copies the block from [the rehydration source](rehydrate.md) into that session |
| Understand what, where, and how Zax documents | Read the [documentation architecture](documentation.md) |
| Conduct a targeted audit of completed work | Read the [archive guidance](archive/README.md) |

Agents receive either the handoff block or the rehydration block in chat. They
must not proactively read either source file during normal work. Read them only
when the language maintainer explicitly asks for their maintenance.

The operating prompt sources mirror the current phase, active work pointer, next
work number, and revision. Maintain them together.

Completed working files are not ordinary reading. Do not inspect
`project/archive/` unless the language maintainer directly requests it or a
targeted audit requires it.
