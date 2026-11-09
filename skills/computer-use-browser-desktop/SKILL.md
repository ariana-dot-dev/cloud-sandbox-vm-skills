---
name: computer-use-browser-desktop
description: Drive a real browser and Linux desktop in the cloud (computer use) — a full GUI with Chrome, a 60fps stream for visibility, and the Lux agent for automating clicks/typing/logins. Use for visual UI checks, logging into web apps, testing Electron apps, or any GUI task that can't be done over plain shell. Records demo videos automatically.
---

# Computer use: browser + desktop (Box)

Use this when a task needs a **real GUI**, not just a shell: logging into a web app, validating a rendered UI, controlling Chrome, testing an Electron app, or recording a demo. Box runs a full Linux desktop with Chrome; the **Lux** agent automates desktop/browser interactions and auto-records the session.

## Recipe

```bash
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"
# Get a live desktop stream URL (60fps WebRTC, or VNC over HTTPS on bad networks)
box desktop "$box_id" --json
# Automate a GUI goal with Lux (records to /home/user/lux-demos/)
box ssh "$box_id" -- "lux start 'log in to example.com and screenshot the dashboard'"
box ssh "$box_id" -- "lux run --max-steps 30"
box ssh "$box_id" -- "lux status"
```

## Why Box

- **Desktop + Chrome built in** — 1920x1080 desktop with `computer` tools for agents, no desktop setup needed. No GPU: graphics run on the CPU.
- **Two stream modes** — 60fps WebRTC (Moonlight) for quality, VNC-over-HTTPS for reliability on restrictive networks.
- **Auto-recorded demos** — every Lux run produces a video artifact.

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Limits

Runs in Germany, Finland and France (no region choice). $20 plan: 100 sandboxes at once (higher plans up to 2,000). For large headless browser fleets, dedicated browser providers (Browserbase) are built for that; Box fits when you need a *full machine* behind the browser.
