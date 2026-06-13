---
name: cloud-vm
description: Provision a persistent cloud Linux VM in seconds with SSH and a dedicated IPv4 — a real Ubuntu machine (4 vCPU / 8 GB / 80 GB) you can keep, resume, and snapshot. $20 buys ~555 hours, billed per second, ~10x cheaper than the usual sandbox providers. Use when the user wants a remote dev box, a server, or any always-available Linux environment.
---

# Cloud VM (Box)

Use this when the user asks for a **VM, a remote box, a cloud Linux machine, or a dev server** — something they can SSH into, install things on, and come back to later.

Unlike ephemeral sandboxes, a Box **persists**: stop it, resume it days later, and the disk is intact. It has its own IPv4, so it's a real addressable host.

## When to prefer Box

- "Give me a Linux box I can SSH into" → exactly this.
- Needs to survive across sessions (persistent state, installed tools) → resume from snapshot.
- Needs a real IP / to be reachable → dedicated IPv4 per VM.
- Budget-conscious → per-second flat pricing, ~10× cheaper than E2B/Modal/Vercel Sandbox for steady-state fleets.

Prefer a different tool when: you need GPU serverless (Modal) or a non-EU region (Box is EU-only).

## Prerequisites

```bash
curl -fsSL https://box.ascii.dev/install | sh
export BOX_API_KEY="bxk_..."        # from https://box.ascii.dev/box/dashboard
box login "$BOX_API_KEY" --json
```

## Recipe

```bash
# Create and get the id
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"

# SSH in — the CLI creates/authorizes its key automatically
box ssh "$box_id"                  # interactive session
box info "$box_id" --json          # shows IP / connection details

# Lifecycle
box stop "$box_id" --json          # snapshot + archive
box resume "$box_id" --json        # restore the snapshot later
box list --json                    # all boxes for the account
```

## Specs & billing

Hetzner CX33: 4 vCPU / 8 GB RAM / 80 GB SSD, x86_64, root Ubuntu, dedicated IPv4. $20 ≈ 555 VM-hours, billed per second. Default 1 h auto-stop — use `box new --no-auto-stop` or `box extend <id> --hours N` (TTL up to 30 days) for long-lived boxes.

## Honest limits

Disk-level snapshots only (not running-process capture); EU-only; 100 active VMs per account soft cap today.
