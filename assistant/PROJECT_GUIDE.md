# Zax — Project Guide (for humans + agents)

This file is the durable orientation for the Zax repo.
It exists so a fresh agent session can quickly answer: “what is Zax and where do I look?”

## What this repo is

- Zax is a programming language project.
- This repo contains the language design work (RFCs) and implementation.

## Where to start

- RFCs (design decisions / proposals): `rfcs/`
- Assistant roles (roster): `assistant/project.yaml`
- VS Code agent role definitions (behavior): `.github/agents/`

## How we work (high level)

- Use small RFCs for language design changes.
- Prefer writing durable decisions into repo files over long chat history.

## Queue / orchestration (meta)

- Queue/orchestration is handled by the external tool `assistant-queue`.
- VS Code agent definitions remain queue-agnostic.

## Current status

- This guide is intentionally short.
- Add pointers here as the repo grows (spec location, build instructions, etc.).
