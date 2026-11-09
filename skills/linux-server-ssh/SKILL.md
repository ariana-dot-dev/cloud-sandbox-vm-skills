---
name: linux-server-ssh
description: Get a real Linux server in the cloud with full SSH/SCP access in seconds — root Ubuntu (default 4 vCPU / 8 GB / 51 GB disk), public IPv6 and outbound IPv4, Docker and every major toolchain preinstalled. $0.036/h billed per second, from a $20/mo plan. Use when the user needs a remote Linux server to SSH into, run services on, or use as a dev machine.
---

# Linux server over SSH (Box)

Use this when the task needs an actual **Linux server** the user can `ssh`/`scp` into: hosting a service, running a daemon, or a remote workstation.

## Why Box

- **First-class SSH** — `box ssh <id>` opens a session; the CLI handles keys. SCP works too. SSH works over IPv4 also on boxes without their own IPv4.
- **Addressable host** — public IPv6 per box, raw TCP/UDP ports through the firewall. The IP changes at each resume; there is no static or dedicated IPv4.
- **Root Ubuntu** — `apt install` anything; Docker preinstalled.
- **Low per-hour price, persistent** — $0.036/h for 4 vCPU / 8 GB vs $0.331/h (E2B) and $0.476/h (Modal) at list price, for servers that run full hours; resumes from snapshot.

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$box_id"                                  # interactive shell
box ssh "$box_id" -- "sudo apt-get install -y nginx && sudo systemctl enable --now nginx"
box info "$box_id" --json                          # shows the IP
```

## Honest limits

Runs in Germany, Finland and France (no region choice); $20 plan: 100 boxes at once (higher plans up to 2,000); disk-level snapshots (running processes don't survive stop/resume). For sub-second boot, see Daytona/E2B/Modal.
