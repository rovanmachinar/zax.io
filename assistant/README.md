# Zax assistant guide (OpenClaw)

This folder exists to keep **one** authoritative, repo-committed definition of the Zax assistant roles and orchestration conventions.

## Source of truth (roles)

- **The one-and-only authoritative roles list:** `assistant/project.yaml`

Do not restate the role list elsewhere in this repo.

## Per-role behavior definitions (authoritative)

The source of truth for **how each role behaves** (prompting, tools, model preferences) is:

- `.github/agents/`

These files must remain **compatible with VS Code agent usage** (one-off runs).

Important constraint:
- Do **not** bake queue/orchestration mechanics into the VS Code agent definitions.
  - VS Code runs these roles as one-off helpers today.
  - Queue-awareness (if/when we add it) belongs in the queue tool and its own docs/config, not in `.github/agents/*`.

The role **labels** in `.github/agents/` must match the labels in `assistant/project.yaml`.

## Orchestration tool (optional)

Zax can also be orchestrated by the external runner **assistant-queue**.

- Runtime state (runs/logs/artifacts/config) should live in a **local, gitignored directory**.
- If you keep any queue-tool state inside this repo, it must be under:
  - `assistant/.assistant-queue/` (gitignored)

The presumption is that `assistant/.assistant-queue/` is safe to delete at any time and can be recreated.

## Queue definitions

If you keep example or committable queues in-repo, store them under:

- `assistant/queues/`

Queues should refer to role labels defined in `assistant/project.yaml`.
