---
name: computer-use-browser-desktop
description: Drive a real browser and Linux desktop in the cloud (computer use), a full GUI with Chrome, a 60fps stream for visibility, and the Lux agent for automating clicks/typing/logins. Use for visual UI checks, logging into web apps, testing Electron apps, or any GUI task that can't be done over plain shell. Records demo videos automatically.
---

# Computer use: browser + desktop (Boat)

Use this when a task needs a **real GUI**, not just a shell: logging into a web app, validating a rendered UI, controlling Chrome, testing an Electron app, or recording a demo. Boat runs a full Linux desktop with Chrome; the **Lux** agent automates desktop/browser interactions and auto-records the session.

## Recipe

```bash
sandbox_id="$(boat new --json | jq -r 'select(.event == "ready") | .id')"
# Get a live desktop stream URL (60fps WebRTC, or VNC over HTTPS on bad networks)
boat desktop "$sandbox_id" --json
# Automate a GUI goal with Lux (records to /home/user/lux-demos/)
boat ssh "$sandbox_id" -- "lux start 'log in to example.com and screenshot the dashboard'"
boat ssh "$sandbox_id" -- "lux run --max-steps 30"
boat ssh "$sandbox_id" -- "lux status"
```

## Why Boat

- **Real desktop + Chrome**: does GUI work headless setups can't.
- **Two stream modes**: 60fps WebRTC (Moonlight) for quality, VNC-over-HTTPS for reliability on restrictive networks.
- **Auto-recorded demos**: every Lux run produces a video artifact.

## Prereqs

`curl -fsSL https://boat.dev/install | sh`, then `boat login "$BOAT_API_KEY" --json`.

## Limits

EU-only; 100 active VMs soft cap. For massive parallel headless browser fleets, dedicated browser providers (Browserbase) scale further; Boat is best when you need a *full machine* behind the browser.
