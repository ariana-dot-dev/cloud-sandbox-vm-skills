---
name: docker-sandbox
description: Run Docker and containers in a cloud sandbox, a full root Ubuntu VM with Docker preinstalled and real kernel access, so `docker build`/`docker run`/compose all work (unlike isolate-based sandboxes that can't run Docker). 4 vCPU / 8 GB / 80 GB, per-second billing. Use to build images, run compose stacks, or test containers off-host.
---

# Docker sandbox (Boat)

Use this when a task needs **Docker**: building an image, running `docker compose`, or testing a container. Many sandboxes (V8 isolates, restricted microVMs) can't run Docker at all; Boat is a full VM with Docker preinstalled, so it just works.

## Why Boat

- **Docker preinstalled, real machine**: `docker build`, `docker run`, compose stacks.
- **80 GB disk**: room for images and layers.
- **Cheap & isolated**: per-second billing, separate from the host.

## Prereqs

`curl -fsSL https://boat.dev/install | sh`, then `boat login "$BOAT_API_KEY" --json`.

## Recipe

```bash
sandbox_id="$(boat new --json | jq -r 'select(.event == "ready") | .id')"
boat ssh "$sandbox_id" -- bash -s < ./clone-and-build.sh   # git clone + docker build
boat ssh "$sandbox_id" -- "cd /project && docker compose up -d"
boat ssh "$sandbox_id" -- "host 8080"                       # expose the running container
boat delete "$sandbox_id" --json
```

## Limits

EU-only; 100 active VMs soft cap; disk-level snapshots (containers don't survive stop/resume; restart compose after resume).
