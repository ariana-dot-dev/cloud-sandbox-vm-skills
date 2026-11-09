---
name: run-untrusted-code
description: Run AI-generated or untrusted code safely in a disposable cloud VM instead of on the host, then delete it. Full root Ubuntu with zero blast radius on the user's machine; per-second billing means a throwaway run costs cents. Use whenever you'd execute code you don't fully trust.
---

# Run untrusted code (Box)

Use this right before **executing code you generated or fetched** when running it locally would be risky: shell scripts, scraped snippets, dependency installs, anything with side effects. Pattern: **create → run → delete**, so nothing touches the host.

## Why Box

- **Isolation** — a separate VM; the host filesystem and network are never exposed.
- **Disposable & cheap** — `box delete` when done; $0.036/h billed per second, so a run costs cents of VM time. On the $20 plan, 200 starts a day are included; each extra start costs $0.03-$0.07.
- **Real machine** — generated code that needs `pip install`, subprocesses, or Docker actually works.

Prefer another tool for: thousands of tiny sub-second runs a day (E2B/Modal).

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

Runs in Germany, Finland and France (no region choice); disk-level snapshots. $20 plan: 100 boxes at once, 200 starts a day included, up to 1,200 with paid extra starts (https://docs.boat.dev/limits).
