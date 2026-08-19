# Zax post-compaction rehydration source

| Field | Value |
| --- | --- |
| Status | Current project guidance |
| Audience | The Zax language maintainer |
| Applies To | Recovering the same design session after context compaction |
| Owns | The paste-ready post-compaction recovery prompt |
| Does Not Own | Documentation architecture, language concepts, or new-session onboarding |

Agents receive the block below in chat. They must not proactively read this
source file except when explicitly assigned to maintain it.

## Post-compaction copy/paste block

The language maintainer may add a short description of where the conversation
had reached. That description is recovery evidence, not edit authorization.

<!-- begin zax rehydration -->

# Zax same-session rehydration

This is the same session after context compaction, not a new project start. Some
operating instructions, discussion state, and authorization boundaries may have
been lost. Recover them before continuing.

## Mirrored current state

| Field | Value |
| --- | --- |
| Operating guidance revision | 2 |
| Current phase | Project guidance established; awaiting preparation of the first numbered work item |
| Active numbered work item | None |
| Next work number | `001` |
| Language-document status | Legacy design input under review; not a formal specification |
| Implementation in this repository | None |

## Purpose and authority

Zax is a compiled, data-oriented programming language intended to span
high-level capabilities and low-level control with explicit cost. It is
presently being refined into a conceptually sound, technically thorough,
human-developer-readable language design. Producing a formal specification and
implementing a compiler or transpiler are future concerns.

Existing language pages, RFCs, examples, plans, summaries, and agent-authored
material are evidence according to their maturity, not automatic authority.
There is no formal Zax specification.

The Zax language maintainer is the language decision authority. An agent may
reason, propose, disagree, identify risks, and trace consequences. It may not
infer alignment, acceptance, or authorization.

## Recovery safety

Do not edit files or mutate version control during this recovery turn.

Discussion always precedes edits. No edit occurs until the language maintainer
and agent discuss the specific change, align on it, and the language maintainer
explicitly authorizes that edit. Alignment and edit authorization are separate.
Prior authorization should be treated as expired when its surviving boundary is
uncertain.

Use ordinary conversational review. Do not use structured, modal,
multiple-choice, MCP, or other tool-based input prompts that force the language
maintainer to choose from a predefined menu.

Editing, staging, committing, and pushing are separate permissions. The language
maintainer may use staging only as a before/after comparison boundary. Staged
content does not imply acceptance, approval, or readiness to commit. Do not
alter that boundary without explicit authorization.

## Recovery procedure

1. Inspect and report the branch and staged/unstaged working-tree state.
2. Read `project/documentation.md`.
3. If Mirrored current state names an active work item, read it.
4. Inspect the current staged and unstaged changes relevant to that work.
5. Reconcile the active file, repository state, latest human instruction, and
   any recovery context supplied in chat.
6. Determine the current phase and commitment point: starting analysis,
   developing findings, awaiting authorization, applying an authorized change,
   performing the pre-promotion dry run, promoting findings, closing the work
   item, or preparing the next item.
7. Report completed work, current authority, unresolved questions, the recovered
   authorization boundary, and the next safe action.
8. Wait for the language maintainer to confirm or correct the recovery before
   continuing.

Do not repeat completed work. Do not infer authorization from summaries, stale
task text, changed files, staged files, silence, or the presence of an active
work item.

The operating guidance has already been supplied in this prompt. Do not read
operating-prompt source files during recovery. Do not read `project/archive/`
unless the language maintainer requests it or a targeted audit requires it.

## Numbered work continuity

There is at most one active numbered concept file. Its fixed initiating input is
an immutable, possibly incomplete anchor. Keep that focus stable while following
material consequences far enough to test coherence. Detailed numbered-work,
pre-promotion, archival, and documentation rules are owned by
`project/documentation.md`.

If recovery shows that item `n` is closing, remember that the cycle must prepare
the aligned initiating input and reading scope for `n+1`. Creating `n+1` does not
authorize beginning it. A new session may take over, or the language maintainer
may explicitly reassign this session after the work-unit boundary.

## Default working style

Use one deep-reasoning agent by default. Explain disagreements and risks plainly.
Prefer concrete programmer use cases. Keep one active concern. Capture adjacent
findings without silently solving them. Helpful analysis is welcome;
unrequested commitment is not.

<!-- end zax rehydration -->
