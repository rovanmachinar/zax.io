# Zax assistant guide (OpenClaw)

This repo is worked on with OpenClaw (Rovan Machinar) plus persistent Zax subagents.

## Where things live

- RFCs: `rfcs/`
- Assistant guide (this file): `assistant/README.md`

## Persistent Zax subagents

These are **project-specific** and keep their own context.

### zax-planner
- Purpose: project planning / RFC drafting / decision tracking.
- Pinned model: `openai/gpt-5.2`
- Use when: turning discussions into small RFCs, tracking open questions.

### zax-researcher
- Purpose: research how other languages solve a design problem; provide options + tradeoffs.
- Pinned model: `openai/gpt-5.2`
- Use when: gathering patterns and references before making a design decision.

### zax-designer
- Purpose: propose concrete language design (syntax + semantics + examples).
- Pinned model: `anthropic/claude-opus-4-6`
- Use when: generating a first-pass design proposal once constraints are clear.

### zax-skeptic
- Purpose: red-team proposals (ambiguities, hazards, long-term costs).
- Pinned model: `anthropic/claude-sonnet-4-5`
- Use when: you want a design challenged before committing to it.

### zax-skeptic-opus
- Purpose: heavyweight/final-pass red-teaming when Sonnet is not enough.
- Pinned model: `anthropic/claude-opus-4-6`
- Use when: you explicitly want the strongest pass and can tolerate tighter rate limits.

### zax-spec-editor
- Purpose: convert accepted RFC decisions into docs/spec/website updates.
- Pinned model: `openai/gpt-5.2`
- Use when: updating documentation after a decision is accepted.

## Routing commands (in the main chat)

Prefix your message with one of these to route it:

- `zax-planner: <prompt>`
- `zax-researcher: <prompt>`
- `zax-designer: <prompt>`
- `zax-skeptic: <prompt>`
- `zax-spec-editor: <prompt>`

## Cancel / status / reset (for when something goes wrong)

Use these in the main chat:

- `cancel-wait` — stop waiting/polling for a subagent reply.
- `cancel-wait zax-planner` — stop waiting for a specific role.
- `status zax-planner` — fetch latest state/output from that session.
- `reset zax-planner` — create a fresh session for that role (old one is retired).

(These are conventions the main assistant follows.)

## Notes on model selection

- Each subagent is **pinned** to a model when created.
- If we need a different model for a one-off, we can spawn a temporary session.

## Persistence across reboots

- Subagent sessions are stored on disk (session transcripts) and should still be visible after restarting OpenClaw.
- If you use a destructive reset of OpenClaw sessions/credentials, sessions can be lost.

## Multiple projects

- Zax roles are specific to this repo.
- For a new project, we will spawn new project-specific roles (for example, `projectname-planner`) and keep them separate.
