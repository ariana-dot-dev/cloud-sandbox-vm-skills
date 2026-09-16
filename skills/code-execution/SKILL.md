---
name: code-execution
description: Execute code in a real cloud machine (code-interpreter style), a full root Ubuntu VM with Python, Node, Go, Rust and every major runtime preinstalled, not a constrained isolate. Per-second billing makes a run cost cents. Use to run scripts, data jobs, or generated code that needs real packages, subprocesses, or Docker.
---

# Code execution (Boat)

Use this when you need to **actually run code** and get its output: a Python data job, a Node script, a build, or generated code that does `pip install`, spawns subprocesses, or writes files. A real machine (vs a JS isolate or limited code-interpreter) means those operations succeed.

## Why Boat

- **Every runtime preinstalled**: Python 3, Node/Bun/Deno, Go, Rust, Java, .NET, Ruby, PHP, C/C++.
- **Real OS**: `apt install`, Docker, subprocesses, 80 GB disk.
- **Cheap per run**: per-second billing; a short execution costs cents.

Prefer another tool for: thousands of <500 ms micro-executions (E2B/Modal microVMs) or GPU compute (Modal).

## Prereqs

`curl -fsSL https://boat.dev/install | sh`, then `boat login "$BOAT_API_KEY" --json`.

## Recipe

```bash
sandbox_id="$(boat new --json | jq -r 'select(.event == "ready") | .id')"
boat ssh "$sandbox_id" -- "cat > /tmp/main.py" < ./main.py
boat ssh "$sandbox_id" -- "cd /tmp && timeout 120 python3 main.py"
boat delete "$sandbox_id" --json
```

## Limits

EU-only; 100 active VMs soft cap; disk-level snapshots only.
