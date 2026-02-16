# RFCs (Zax)

This folder contains lightweight design documents (“RFCs”) for the Zax language and related tooling.

## Location
- All RFCs live in: `rfcs/`

## Naming convention
Use: `YYYY-MM-DD-<slug>.md`

Example:
- `rfcs/2026-02-16-structural-typing-equivalence.md`

## Status lifecycle
Each RFC should include a small header with a `Status:` line:

- **Draft** — under discussion / incomplete.
- **Accepted** — approved direction; implementation may follow in separate PRs.
- **Rejected** — considered and declined (keep for history).
- **Superseded** — replaced by a newer RFC (link to the successor).

When an RFC changes status, update the `Status:` line and add a short note/link in the RFC (or in git history via the PR).

## Recording decisions
When an RFC is Accepted/Rejected/Superseded, record the decision with:
- a link to the **merged PR** that made the status change (preferred), or
- the **commit hash** if no PR exists.

Recommended: add a `Related:` line in the RFC header pointing to the PR/commit.

## Tiny example entry
```markdown
- RFC: 2026-02-16-structural-typing-equivalence
  - File: rfcs/2026-02-16-structural-typing-equivalence.md
  - Status: Draft
  - Related: (add PR link when decided)
```
