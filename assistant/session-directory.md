# Session directory (local debugging)

This file is for Robin’s convenience when debugging OpenClaw session routing.

- It is **not authoritative**.
- Session keys/IDs may change after resets.
- Prefer using OpenClaw’s UI/session list when available.

## Zax persistent subagents

(These keys/IDs are informational and may change after resets.)

- zax-planner
  - sessionKey: agent:main:subagent:9d3e97ec-18c1-42bb-89dc-47fb927b1077
  - sessionId: 549ae3d4-e02c-4de8-9753-e599fd0a4dfc
  - model: openai/gpt-5.2

- zax-researcher
  - sessionKey: agent:main:subagent:7bd40d50-123e-43c1-9731-2aababd2fffe
  - sessionId: 86e93a8b-19ac-466b-a2a2-9dab90a09369
  - model: openai/gpt-5.2

- zax-spec-editor
  - sessionKey: agent:main:subagent:0da9f951-a2b9-4dbe-afc2-cb2de0b6cff7
  - sessionId: 8bb8173f-8741-4e7d-b668-222b2649fbea
  - model: openai/gpt-5.2

- zax-designer
  - sessionKey: agent:main:subagent:f2d68332-73d4-4015-aaa0-87eb1c805ef1
  - sessionId: cff04f3b-43ff-4ae8-9444-bec09ee53042
  - model: anthropic/claude-opus-4-6

- zax-skeptic (primary)
  - sessionKey: agent:main:subagent:8e746532-7017-4c04-929b-afa01dfdad8f
  - sessionId: 99847a75-8e61-43d5-9f47-afbad00b3520
  - model: anthropic/claude-sonnet-4-5

- zax-skeptic-opus (heavyweight / final-pass)
  - sessionKey: agent:main:subagent:1c3c9efb-e5c3-4ea1-8a6f-da0e9c8e6bbd
  - sessionId: 9b5530d3-2d30-4036-a60a-997e3ee2de26
  - model: anthropic/claude-opus-4-6

## Retired / superseded

- zax-skeptic (old; misconfigured during Sonnet enablement)
  - sessionKey: agent:main:subagent:aecd3c25-e2a0-4940-b798-fac4bea08f47
  - sessionId: bd87caa6-c288-42cd-9cd8-36a8213f6a4c
  - model: openai/gpt-5.2
  - note: keep for forensic reference; do not use.

## Notes

If a session appears in the gateway dashboard only as a GUID, identify it by:
- last message content
- model/provider
- or by searching transcripts under:
  - `/home/rovan/.openclaw/workspace/*.jsonl`
