# zax.io — AI-friendly repo plan (docs + structure, no semantics changes)

**Intent:** Make this repo easier for OpenClaw subagents and VS Code agents to navigate, reason about, and contribute to **without changing language semantics yet**.

**Format:** Practical checklist, split into small PRs (PR0, PR1, PR2, …). Prioritize quick wins.

---

## Why this helps agents (short)

- **Faster orientation:** one obvious entrypoint + docs index + repo map.
- **Less hallucination:** glossary + explicit “normative vs descriptive” doc routing.
- **Safer contributions:** contribution rules + RFC process + public-safe policy.
- **Less thrash:** clear status markers for incomplete/broken/unknown areas + known-issues ledger.

---

## Principles (apply everywhere)

- **One obvious entrypoint:** `README.md` points to exactly where to start.
- **Single source of truth:** label what’s *normative* (spec) vs *descriptive* (architecture) vs *process* (RFC/contributing).
- **Progressive disclosure:** short overview pages that link deeper.
- **Explicit status:** every doc has `Status: Stable|Draft|Incomplete|Broken|Unknown`.
- **Task-oriented routing:** “If you want to X, read Y; then do Z.”
- **Public-safe by default:** no private/internal references, codenames, internal URLs, credentials, or customer data.

---

# PR0 — Docs scaffolding baseline (quick win)

**Goal:** Create minimal structure + routing so an agent finds the right artifact in <60s.

### Checklist

- [ ] **Update `README.md`**
  - [ ] Add **Start here** section (1–3 sentences: what zax.io is; what’s in-repo).
  - [ ] Add links:
    - [ ] Docs Index: `docs/README.md`
    - [ ] Assistant guide: `ASSISTANT_GUIDE.md`
    - [ ] Contributing (placeholder link for PR1): `CONTRIBUTING.md`
    - [ ] Spec index (placeholder link for PR2): `docs/spec/README.md`
    - [ ] Architecture index (placeholder link for PR3): `docs/architecture/README.md`
  - [ ] Add a one-liner: **All docs are intended to be public-safe.**

- [ ] **Add `docs/README.md` (Docs Index)**
  - [ ] Table of contents with short descriptions.
  - [ ] Add a **Status table** for each doc (Stable/Draft/Incomplete/etc.).
  - [ ] Include “Where to start” for:
    - [ ] Understanding the language (spec)
    - [ ] Understanding codebase (architecture)
    - [ ] Making changes (contributing + RFC)

- [ ] **Add `ASSISTANT_GUIDE.md` (repo root)**
  - [ ] “If you only read one thing, read this.”
  - [ ] What agents should do first.
  - [ ] Routing rules:
    - [ ] Language behavior questions → `docs/spec/*` (normative when Stable)
    - [ ] Code structure questions → `docs/architecture/*` (descriptive)
    - [ ] Proposing changes → `docs/RFCs/*` + `CONTRIBUTING.md`
  - [ ] Hard boundaries:
    - [ ] **No semantics changes** in this doc/structure effort.
    - [ ] Prefer small PRs; avoid broad refactors.

- [ ] **Add `docs/STATUS.md`**
  - [ ] Define allowed status tags and what each means.
  - [ ] Define a standard doc header block (copy/paste):
    - [ ] `Status:`
    - [ ] `Last reviewed:`
    - [ ] `Owner:` (optional; keep public-safe)
    - [ ] `Confidence:` (optional)

- [ ] **Add `docs/PUBLIC_SAFE.md`**
  - [ ] Clear rules: no internal codenames, no internal URLs, no credentials/tokens, no customer data.
  - [ ] “When in doubt, generalize or omit.”

### Marking incomplete/broken areas (adopt now)

- [ ] Add doc header block to new docs:
  - `Status: Incomplete` (default for new skeleton docs)
  - `Last reviewed: YYYY-MM-DD`
- [ ] Use grep-able inline markers:
  - [ ] `TODO(docs):`
  - [ ] `TODO(spec):`
  - [ ] `UNKNOWN:`
  - [ ] `BROKEN:` (known-wrong statements)

---

# PR1 — Contribution guide + RFC process routing

**Goal:** Make contribution flow safe for agents; clarify when to use an RFC vs direct PR.

### Checklist

- [ ] **Add/Update `CONTRIBUTING.md`**
  - [ ] Scope rules: doc-only vs implementation PRs.
  - [ ] Explicit: **No language semantics changes** unless an RFC says so (future).
  - [ ] Small-PR guidance (one theme per PR; suggest size target).
  - [ ] Review expectations and checklists.
  - [ ] Public-safe checklist (link to `docs/PUBLIC_SAFE.md`).
  - [ ] PR title/commit conventions (if you have them).

- [ ] **Add `docs/RFCs/README.md` (RFC index + lifecycle)**
  - [ ] Define states: Proposed / Accepted / Implemented / Deprecated.
  - [ ] Define when RFC is required vs optional.

- [ ] **Add `docs/RFCs/0000-template.md`**
  - [ ] Problem statement
  - [ ] Goals / Non-goals
  - [ ] Proposed design
  - [ ] Alternatives
  - [ ] Compatibility / migration
  - [ ] Security / public-safe notes
  - [ ] Open questions

- [ ] **Update routing links**
  - [ ] `README.md` links to `CONTRIBUTING.md` + `docs/RFCs/README.md`.
  - [ ] `ASSISTANT_GUIDE.md` includes a “Should I write an RFC?” decision section.

### Marking incomplete/broken areas

- [ ] RFC index entries include state + short summary.
- [ ] Contribution guide describes how to label `broken`/`unknown` in docs and how to file issues (if applicable).

---

# PR2 — Spec skeleton (normative home, mostly placeholders)

**Goal:** Create a canonical place for the language spec, even if initially incomplete. **Do not change semantics**; only create scaffolding and clearly label it.

### Checklist

- [ ] **Add `docs/spec/README.md` (Spec index)**
  - [ ] Explain what is **normative** vs “notes”.
  - [ ] Add “Known gaps” section.
  - [ ] Link to each spec chapter.
  - [ ] Set expectations: not complete until marked Stable.

- [ ] **Add skeleton spec chapters** (headers + TODOs only)
  - [ ] `docs/spec/lexing.md`
  - [ ] `docs/spec/parsing.md`
  - [ ] `docs/spec/types.md`
  - [ ] `docs/spec/runtime.md` (if applicable)
  - [ ] `docs/spec/stdlib.md`
  - [ ] `docs/spec/errors.md`
  - [ ] `docs/spec/compatibility.md` (versioning, stability, future policy)

- [ ] **Add status blocks to each spec doc**
  - [ ] `Status: Incomplete`
  - [ ] `Last reviewed: YYYY-MM-DD`
  - [ ] Optional `Confidence: Low`

- [ ] **Update docs index**
  - [ ] `docs/README.md` includes Spec section + statuses.

---

# PR3 — Architecture overview + repo map

**Goal:** Give agents a codebase map and high-level dataflow without making shaky normative claims.

### Checklist

- [ ] **Add `docs/architecture/README.md`**
  - [ ] Index + status table.

- [ ] **Add `docs/architecture/overview.md`**
  - [ ] Components/modules list.
  - [ ] High-level pipeline (parse → analyze → …) as *observed*.
  - [ ] Note where behavior is uncertain.

- [ ] **Add `docs/architecture/repo-map.md`**
  - [ ] Directory-by-directory explanation.
  - [ ] “Where to find X” (parser, AST, tests, CLI, etc.).

- [ ] **Optional: `docs/architecture/diagrams.md`**
  - [ ] Mermaid/text diagrams if allowed.

### Marking incomplete/broken areas

- [ ] Add `Confidence: High|Medium|Low` to architecture sections derived from code reading.
- [ ] Include “Not yet verified” subsection instead of guessing.

---

# PR4 — Glossary + terminology rules

**Goal:** Stop agents from inventing terminology; unify naming across docs.

### Checklist

- [ ] **Add `docs/GLOSSARY.md`**
  - [ ] Common terms and preferred meanings.
  - [ ] Tag entries as Stable/Draft if needed.

- [ ] **Link glossary everywhere**
  - [ ] `README.md`
  - [ ] `docs/README.md`
  - [ ] `docs/spec/README.md`
  - [ ] `docs/architecture/README.md`

---

# PR5 — Known issues / rough edges ledger

**Goal:** Provide a canonical place for broken docs, spec gaps, and implementation rough edges so agents don’t rediscover them.

### Checklist

- [ ] **Add `docs/KNOWN_ISSUES.md`**
  - [ ] Sections: Docs gaps / Spec unknowns / Implementation rough edges (descriptive).
  - [ ] Each entry includes: pointer, impact, suggested next step (doc/test/RFC).

- [ ] **Optional: issue labels (if GitHub issues are used)**
  - [ ] `docs`, `spec`, `architecture`, `broken-doc`, `good-first-doc`, `needs-redaction`

---

# PR6 — Assistant workflows + task recipes

**Goal:** Give agents bounded playbooks for common tasks.

### Checklist

- [ ] **Expand `ASSISTANT_GUIDE.md`**
  - [ ] Task recipes:
    - [ ] Add a doc section properly
    - [ ] Update spec skeleton safely
    - [ ] Trace feature behavior (link code pointers only)
    - [ ] Write an RFC
  - [ ] “Do not” list:
    - [ ] Don’t change semantics without RFC
    - [ ] Don’t rename public interfaces
    - [ ] Don’t add dependencies unless requested

- [ ] **Optional: `docs/assistant/`**
  - [ ] `docs/assistant/workflows.md`
  - [ ] `docs/assistant/checklists.md`

---

## Standard sections

### Assumptions

- Repo documentation is intended to be **public-safe**.
- The immediate goal is agent navigation + contribution safety, not correctness/completeness of the spec.
- No language semantics changes are desired in this workstream.

### Out of scope (for now)

- Behavior/semantics changes, refactors, renames of public APIs.
- CI enforcement (link checking, doc lint, banned-term scanning) beyond documenting policies.
- Building a full docs website (can come later).

### Risks & mitigations

- **Risk:** Skeleton spec is misread as authoritative.
  - **Mitigation:** Prominent `Status: Incomplete` and “not normative until Stable” messaging in `docs/spec/README.md`.
- **Risk:** Private/internal references leak into public docs.
  - **Mitigation:** `docs/PUBLIC_SAFE.md` + contributing checklist + `needs-redaction` review label.
- **Risk:** Docs drift quickly.
  - **Mitigation:** Status blocks + `Last reviewed` + `docs/KNOWN_ISSUES.md` ledger.

### Success criteria

- A new agent can locate spec/architecture/process docs in <60 seconds.
- Reduced duplicate questions (“where is X?”) and fewer speculative changes.
- Small PRs are easy to review and merge.

### Open questions

- Do you want RFCs PR-based only (files in repo) or also issue-based?
- Should the spec be Markdown-only long-term, or later published via MkDocs/Docusaurus?
- Any known “messy” directories that should be explicitly flagged in the repo map?
