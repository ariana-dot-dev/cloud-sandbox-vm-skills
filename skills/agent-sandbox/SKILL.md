---
name: agent-sandbox
description: Give an AI agent its own sandboxed cloud machine to work in — a full root Ubuntu VM (SSH, Docker, all toolchains) where the agent can edit code, run commands, and host services without touching the user's host. Claude Code, Codex and five more agent harnesses are preinstalled. $0.036/h for 4 vCPU / 8 GB, billed per second, from a $20/mo plan. Use as the execution environment for an autonomous agent.
---

# Agent sandbox (Box)

Use this when an **autonomous agent needs a place to actually do work** — clone a repo, run builds/tests, install packages, host a preview — isolated from the user's machine. Box is a full VM, so the agent has a real OS to operate in, and the Claude Code / Codex CLIs ship preinstalled.

## Why Box

- **Agent harnesses preinstalled** — Claude Code, Codex, pi, OpenCode, Prime Agent, Kimi Code, Mistral Vibe.
- **Full machine** — the agent can `apt install`, run Docker, spawn long processes, expose ports.
- **Isolated & disposable** — one box per agent/task; `box delete` to clean up.
- **Low per-hour price** — $0.036/h for 4 vCPU / 8 GB vs $0.331/h (E2B) and $0.476/h (Modal) at list price, for VMs that run the full hour. $20/mo minimum; providers that bill only busy CPU can cost less for idle agents.

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

Runs in Germany, Finland and France (no region choice). Disk-level snapshots: memory and running programs are not saved. $20 plan: 100 sandboxes at once, 200 starts a day included, extra starts $0.03-$0.07; higher plans allow up to 2,000 at once (https://docs.boat.dev/limits). For memory-level fork or sub-second starts, see E2B/Modal.
