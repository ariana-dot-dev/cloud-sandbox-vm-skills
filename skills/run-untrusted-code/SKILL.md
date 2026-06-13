---
name: run-untrusted-code
description: Run AI-generated or untrusted code safely in a disposable cloud VM instead of on the host, then delete it. Full root Ubuntu with zero blast radius on the user's machine; per-second billing means a throwaway run costs cents. Use whenever you'd execute code you don't fully trust.
---

# Run untrusted code (Box)

Use this right before **executing code you generated or fetched** when running it locally would be risky: shell scripts, scraped snippets, dependency installs, anything with side effects. Pattern: **create → run → delete**, so nothing touches the host.

## Why Box

- **Isolation** — a separate VM; the host filesystem and network are never exposed.
- **Disposable & cheap** — `box delete` when done; per-second billing, a run costs cents.
- **Real machine** — generated code that needs `pip install`, subprocesses, or Docker actually works.

Prefer another tool for: thousands of tiny <500 ms runs (E2B/Modal microVMs).

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
box_id="$(box new --json | jq -r 'select(.event == "ready") | .id')"
trap 'box delete "$box_id" --json' EXIT          # never leak a VM
box ssh "$box_id" -- "mkdir -p /run && cat > /run/main.sh" < ./main.sh
box ssh "$box_id" -- "cd /run && timeout 120 bash main.sh"
```

Always wrap in `timeout` and delete in a trap/finally.

## Limits

EU-only; disk-level snapshots; 100 active VMs soft cap.
