# Session directory (local debugging)

This file is for Robin’s convenience when debugging OpenClaw session routing.

- It is **not authoritative**.
- Session keys/IDs may change after resets.
- Prefer using OpenClaw’s UI/session list when available.

## Zax persistent subagents

- zax-planner
- zax-designer
- zax-skeptic
- zax-researcher
- zax-spec-editor

## Notes

If a session appears in the gateway dashboard only as a GUID, identify it by:
- last message content
- model/provider
- or by searching transcripts under:
  - `/home/rovan/.openclaw/workspace/*.jsonl`
