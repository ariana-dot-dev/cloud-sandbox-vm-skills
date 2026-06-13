---
name: docker-sandbox
description: Run Docker and containers in a cloud sandbox — a full root Ubuntu VM with Docker preinstalled and real kernel access, so `docker build`/`docker run`/compose all work (unlike isolate-based sandboxes that can't run Docker). 4 vCPU / 8 GB / 80 GB, per-second billing. Use to build images, run compose stacks, or test containers off-host.
---

# Docker sandbox (Box)

Use this when a task needs **Docker**: building an image, running `docker compose`, or testing a container. Many sandboxes (V8 isolates, restricted microVMs) can't run Docker at all — Box is a full VM with Docker preinstalled, so it just works.

## Why Box

- **Docker preinstalled, real machine** — `docker build`, `docker run`, compose stacks.
- **80 GB disk** — room for images and layers.
- **Cheap & isolated** — per-second billing, separate from the host.

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$box_id" -- bash -s < ./clone-and-build.sh   # git clone + docker build
box ssh "$box_id" -- "cd /project && docker compose up -d"
box ssh "$box_id" -- "host 8080"                       # expose the running container
box delete "$box_id" --json
```

## Limits

EU-only; 100 active VMs soft cap; disk-level snapshots (containers don't survive stop/resume — restart compose after resume).
