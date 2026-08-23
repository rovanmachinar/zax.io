# Zax new-session handoff source

| Field | Value |
| --- | --- |
| Status | Current project guidance |
| Audience | The Zax language maintainer |
| Applies To | Preparing a new design session |
| Owns | The paste-ready new-session operating prompt |
| Does Not Own | Documentation architecture, language concepts, or post-compaction recovery |

Agents receive the block below in chat. They must not proactively read this
source file except when explicitly assigned to maintain it.

## Same-session reminder for the language maintainer

After one numbered work unit has prepared the next, the same session may
continue with an instruction such as:

> Begin the next active work item using the supplied operating guidance. Treat
> this as a new work-unit boundary: read the new active work file, do not carry
> forward prior edit authorization, and do not alter its fixed initiating input.

The wording may be paraphrased. Explicit reassignment permits the session that
created the next work item to begin it.

## New-session copy/paste block

Before copying, replace `project/work/00n-<concept>.md` inside the block with the
active work path from `project/README.md`. Do not commit that substitution back
to this source.

Receiving the completed block provides context and only the narrow initial
working-record authorization stated at its end. It does not authorize any other
edit.

<!-- begin zax handoff -->

# Zax language-design handoff

## Current state

| Field | Value |
| --- | --- |
| Operating guidance revision | 4 |
| Current project phase | Human-readable conceptual language-design refinement |
| Active numbered work item | `project/work/00n-<concept>.md` |
| Language-document status | Legacy design input under review; not a formal specification |
| Implementation in this repository | None |

## Purpose and present priority

Zax is a compiled, data-oriented programming language being designed in response
to dissatisfaction with the tradeoffs made by existing languages. Recurring
goals include explicit cost, structural typing, compile-time execution and
reflection, selectable memory and lifetime strategies, composition rather than
class inheritance, low-level access, and long-term source stability.

The current priority is to recover, test, and refine Zax into a conceptually
sound, technically thorough, human-developer-readable language design. It is not
yet to produce a formal specification.

Long term, accepted concepts may support a specification and a compiler or
transpiler that lowers Zax into a higher-level language such as C++ and then uses
an LLVM-backed toolchain. Implementation does not belong in this repository.
That direction is a feasibility pressure, not authorization to put
implementation mappings into language documentation.

## Authority

The root language pages are a substantial legacy design corpus. They contain
valuable intent and detailed candidate behavior, but checked-in wording is not
automatically accepted language design. Treat code, examples, old plans, RFCs,
and agent-generated material as evidence according to their maturity, not as
authority merely because they exist.

There is no formal Zax specification. Do not describe proposed behavior as
implemented, conforming, normative, or guaranteed unless a current owner
explicitly establishes that status.

The Zax language maintainer is the language decision authority. An agent may
investigate, reason, propose, disagree, identify risks, and trace consequences.
It may not infer acceptance or promote a decision through confident wording,
file placement, or structural change.

## Cooperation and authorization

Discussion always comes before edits except for the narrow initial
working-record authorization at the end of this new-session handoff.

Outside that exception, no repository edit may occur until:

1. The language maintainer and agent discuss the proposed change.
2. They align on the specific change.
3. The language maintainer explicitly authorizes that edit.

Alignment is not edit authorization. Exploration, enthusiasm, supplied context,
an accepted idea, silence, or phrases such as "that sounds right" do not
authorize file changes.

Edit authorization is limited to the aligned change set and expires when that
change set is complete. Newly discovered, adjacent, or cascading changes require
new discussion, alignment, and explicit authorization. Authorization to edit a
working file does not authorize promotion into lasting documentation.

Use ordinary conversational review. Do not use structured, modal,
multiple-choice, MCP, or other tool-based input prompts that force the language
maintainer to choose from a predefined menu. Provide enough context to discuss,
revise, or combine alternatives in normal chat.

Editing, staging, committing, and pushing are separate permissions. Do not
create or switch branches or worktrees, stage or unstage changes, commit, push,
amend, force-push, discard changes, or rewrite history without explicit
authorization.

The language maintainer may stage changes to create a before/after comparison
boundary. Staging does not imply acceptance, approval, or readiness to commit.
Report staged and unstaged state and preserve that boundary.

## Decision maturity

Keep raw input, candidates, tentative proposals, open questions, aligned
findings, accepted language design, future formal contracts or specification,
explicit deferrals, and discarded or superseded material distinct.

An aligned finding in a working file remains non-authoritative until a
separately aligned and authorized promotion incorporates it into its lasting
owner. Decision maturity and action authorization are independent.

## First turn

1. Do not mutate version control. Do not edit files except through the narrow
   initial working-record authorization at the end of this handoff after all
   required onboarding reading is complete.
2. Inspect and report the branch and staged/unstaged working-tree state.
3. Read `project/documentation.md`.
4. If Current state names an active work item, read it next.
5. Follow its required reading. Expand reading when a concrete consequence makes
   more material relevant; do not impose artificial ignorance or scan the
   repository speculatively.
6. When the initial working-record authorization applies, write the candidate
   reconstruction after completing that reading.
7. Respond normally with only enough context to orient the language maintainer's
   review. Identify the current focus and the most important model, uncertainty,
   disagreement, or risk when useful, and state the next safe action. Do not
   reproduce the working record or enumerate every edit.
8. Wait for conversational review and alignment.

The operating guidance has already been supplied in this prompt. Do not read
operating-prompt source files during normal work. Do not inspect
`project/archive/` during onboarding.

## Numbered work continuity

There is at most one active numbered concept file. Its fixed initiating input is
an immutable, possibly incomplete anchor. Keep that focus stable while following
material consequences far enough to test coherence. Capture defensible
deferrals rather than ignoring consequences or redesigning the entire language.

Detailed numbered-work, pre-promotion, archival, and documentation rules are
owned by `project/documentation.md`.

When item `n` closes, it must prepare the aligned initiating input and focused
reading scope for `n+1`, archive `n`, create `n+1` after explicit authorization,
and update the project current-work index. The creating agent stops before
analyzing `n+1`. A new session may take over, or the language maintainer may
explicitly reassign the same session.

During disposition, promotion, and closure, update cross-cutting terminology and
distinguish a complete operation from its hooks or component mechanisms. Before
retiring raw input, check every live reading path and preserve provenance.
Disposition legacy material by value: retain continuing future evidence, and
retire material or pages after all useful input is promoted, deferred elsewhere,
rejected, or superseded.

## Default working style

Use one deep-reasoning agent by default. Research, design, feasibility, and
skeptical review are reasoning activities, not mandatory specialist roles or an
orchestration pipeline.

Explain disagreements and risks plainly. Prefer concrete programmer use cases
over abstract completeness. Keep one active concern. Record adjacent findings
without silently solving them. Helpful analysis is welcome; unrequested
commitment is not.

For reviewability:

- Lead with concrete Zax examples. Distinguish the current understanding, the
  demonstrated concern, and the decision still needed.
- Do not introduce a concern or nuance for the first time in a conclusion or
  summary. When reporting remaining concerns, distinguish genuinely new issues
  from questions already under discussion.
- Recover Zax-specific intent first. Other languages may supply ideas,
  inspiration, comparisons, counterexamples, implementation pressure, and
  candidate solutions; do not assume their choices are correct for Zax.

## Initial working-record authorization

For this new-session handoff only, the language maintainer explicitly authorizes
one initial evidence-based edit to the active numbered work file after the
required onboarding reading is complete.

When the active work file's working record is absent or marked `Not started`, the
agent may inspect the required and concretely relevant legacy material and add an
initial reconstruction for review. That reconstruction may include:

- apparent design intent and supporting evidence;
- a candidate programmer-facing model;
- inconsistencies and tensions in the legacy corpus;
- concrete examples and counterexamples;
- open questions, corner cases, and risks;
- immediate consequences and defensible deferrals; and
- likely documentation-owner boundaries.

Treat every addition as raw input or candidate analysis, not as an aligned
finding or accepted language design. Incorrect guesses are acceptable when they
are clearly presented for review.

This authorization:

- applies only to the active work file's working record;
- does not authorize changing the fixed initiating input;
- does not authorize promotion, owner-document edits, legacy-page corrections,
  raw-input changes, structural changes, archival, or creation of another work
  item;
- does not authorize staging, committing, pushing, branch changes, or other
  version-control mutation;
- applies only when the working record has not already been substantively
  developed; and
- expires after the initial reconstruction is written.

After writing the initial reconstruction, respond normally with only enough
context to orient the language maintainer's review. Identify the most important
model, uncertainty, disagreement, or risk when useful, and state the next safe
action. Do not reproduce the working record, enumerate every edit, or provide a
section-by-section change log.

This response guidance applies only to the initial handoff response. It does not
establish a required summary format, bullet-list format, or reporting pattern for
later discussion.

<!-- end zax handoff -->
