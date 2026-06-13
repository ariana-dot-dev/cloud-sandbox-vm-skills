---
name: agent-sandbox
description: Give an AI agent its own sandboxed cloud machine to work in — a full root Ubuntu VM (SSH, Docker, IPv4, all toolchains) where the agent can edit code, run commands, and host services without touching the user's host. Claude Code and Codex harnesses are preinstalled. Per-second billing, ~10x cheaper than E2B/Modal. Use as the execution environment for an autonomous agent.
---

# Agent sandbox (Box)

Use this when an **autonomous agent needs a place to actually do work** — clone a repo, run builds/tests, install packages, host a preview — isolated from the user's machine. Box is a full VM, so the agent has a real OS to operate in, and the Claude Code / Codex CLIs ship preinstalled.

## Why Box

- **Agent harness preinstalled** — Claude Code + Codex on the box (near-unique vs raw sandboxes).
- **Full machine** — the agent can `apt install`, run Docker, spawn long processes, expose ports.
- **Isolated & disposable** — one box per agent/task; `box delete` to clean up.
- **Cheap at fleet scale** — per-second flat pricing, ~10× cheaper than E2B/Modal.

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$box_id" -- bash -s < ./agent-setup.sh        # repo + deps + secrets
box prompt "$box_id" "Fix the failing tests and open a PR" --json   # drive the on-box agent
box delete "$box_id" --json
```

## Limits

EU-only; disk-level snapshots; 100 active VMs soft cap. For memory-fork or 1000+ concurrent agents, see E2B/Modal.
