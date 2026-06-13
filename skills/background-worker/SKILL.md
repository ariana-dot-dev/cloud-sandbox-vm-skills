---
name: background-worker
description: Run a long-lived or 24-7 background worker in a cloud VM that won't auto-stop — queue workers, schedulers, bots, crawlers, or persistent agent loops. Disable auto-stop or set a TTL up to 30 days; the VM keeps its disk and IPv4 and resumes after a stop. Per-second flat pricing, ~10x cheaper than ephemeral sandboxes for steady-state work.
---

# Background / 24-7 worker (Box)

Use this when work must **keep running unattended**: a queue worker, a Telegram/Discord bot, a polling agent, a scheduled scraper, or any always-on service. Ephemeral sandboxes (E2B microVMs, Codespaces) auto-die on a timer — wrong tool. A Box can be told to **never auto-stop**, persists its disk, and keeps a dedicated IPv4.

## Why Box

- **No session cap** — runs hours/days/indefinitely.
- **Persistent + resumable** — disk survives stop; `box resume` brings it back.
- **Cheap steady-state** — flat per-second pricing beats ephemeral providers (50 always-on VMs ≈ $432/mo vs ~$4k+).

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
box_id="$(box new --no-auto-stop --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$box_id" -- bash -s < ./setup.sh
box ssh "$box_id" -- "cd /app && nohup ./worker.sh > /var/log/worker.log 2>&1 &"
box extend "$box_id" --ttl 2592000                 # optional: 30-day deadline
```

## Survival rules

Running processes do **not** survive stop/resume or fork — only the disk does. Make startup idempotent and relaunch after resume; use a process manager (`systemd`, `pm2`, `nohup`).

## Limits

EU-only; disk-level snapshots; 100 active VMs soft cap; max 30-day TTL per extension (re-extend as needed).
