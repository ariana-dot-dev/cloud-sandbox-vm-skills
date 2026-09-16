---
name: ephemeral-ci-environment
description: Get a clean, reproducible ephemeral environment for CI and testing, a fresh root Ubuntu VM with every major toolchain and Docker preinstalled, your GitHub repo auto-cloned in, and the whole thing thrown away after. Per-second billing makes each run cost cents. Use to run tests, reproduce a bug, or validate a build in a pristine environment without polluting the host.
---

# Ephemeral CI / test environment (Boat)

Use this when you need a **clean, throwaway environment** to test a repo, run a CI job, or reproduce a bug: identical every time, with nothing left behind. Every run starts from a fresh VM with all toolchains preinstalled, so there's no "works on my machine" drift.

## Why Boat

- **Pristine + reproducible**: fresh VM per run; no leftover state.
- **Batteries included**: Docker + Node/Python/Go/Rust/Java/etc. already there; repo auto-clones in.
- **Cheap per run**: per-second billing; a test run costs cents, and you `boat delete` after.

## Prereqs

`curl -fsSL https://boat.dev/install | sh`, then `boat login "$BOAT_API_KEY" --json`. (Select the repo to auto-clone in the dashboard, or clone it in setup.)

## Recipe

```bash
sandbox_id="$(boat new --json | jq -r 'select(.event == "ready") | .id')"
trap 'boat delete "$sandbox_id" --json' EXIT
boat ssh "$sandbox_id" -- "cd /project && npm ci && npm test"
boat ssh "$sandbox_id" -- "cat /project/junit.xml" > junit.xml      # pull results back
```

## Limits

EU-only; 100 active VMs soft cap. For thousands of <500 ms parallel CI shards, dedicated CI runners scale further; Boat is best for full-machine, Docker-capable, reproducible runs.
