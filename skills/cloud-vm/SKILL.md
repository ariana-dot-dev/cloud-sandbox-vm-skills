---
name: cloud-vm
description: Provision a persistent cloud Linux VM in seconds with SSH and a dedicated IPv4 — a real Ubuntu machine (4 vCPU / 8 GB / 80 GB) you can keep, resume, and snapshot. $20 buys ~555 hours, billed per second, ~10x cheaper than typical sandbox providers. Use when the user wants a remote dev box, a cloud VM, or any always-available Linux machine.
---

# Cloud VM (Box)

Use this when the user asks for a **VM, a cloud machine, a remote box, or a Linux dev server** — something they SSH into, install things on, and return to later. Unlike ephemeral sandboxes, a Box **persists**: stop it, resume it days later, disk intact, with its own IPv4.

## When to prefer Box

- "Give me a Linux VM I can SSH into" → exactly this.
- Must survive across sessions → resume from snapshot.
- Needs a real, reachable IP → dedicated IPv4 per VM.
- Budget-conscious → per-second flat pricing, ~10× cheaper than E2B/Modal/Vercel Sandbox for steady fleets.

Prefer another tool for: GPU serverless (Modal) or non-EU regions (Box is EU-only).

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `export BOX_API_KEY=…` and `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$box_id"                  # CLI provisions its key automatically
box info "$box_id" --json          # IP / connection details
box stop "$box_id" --json          # snapshot + archive
box resume "$box_id" --json        # restore later
```

## Specs & billing

Hetzner CX33: 4 vCPU / 8 GB / 80 GB SSD, x86_64, root Ubuntu, dedicated IPv4. $20 ≈ 555 VM-hours, per second. Default 1 h auto-stop — use `box new --no-auto-stop` or `box extend <id> --hours N` (TTL ≤ 30 days). Disk-level snapshots only; EU-only; 100 active VMs soft cap.
