---
name: parallel-agents
description: Build an "agent factory" — configure one cloud VM once, then disk-level fork it into a fleet of identical isolated machines so many agents run in parallel, each with its own filesystem, IPv4, and SSH. Flat per-second pricing makes a fleet ~10x cheaper than E2B/Modal (50 VMs ≈ $432/mo vs ~$4k). Use to fan work across many real machines instead of threads.
---

# Parallel agents / agent factory (Box)

Use this when work should be **fanned out across many isolated environments**: N agents on N tasks, a config sweep, or sharding a workload — and each worker needs a *real* machine, not a coroutine. Box move: configure **one** box (clone, install, secrets), then **fork** it; each fork is an independent VM from that snapshot, so you skip re-provisioning every machine.

## Why Box

- **Fork = a configured template** — one setup, N identical machines (disk-level, no re-clone per VM).
- **True isolation per agent** — separate filesystem, dedicated IPv4, SSH; agents can even talk over the network.
- **Fleet economics** — 50 VMs × 240 h/mo ≈ **$432 (Box)** vs ~$4,124 (E2B) / ~$7,160 (Modal).

## Honest scoping

Box forks at the **disk** level, not memory — a fork does **not** keep the parent's running processes; re-run your start command after fork. For *running-process* fork or routinely **1000+ concurrent** VMs with <500 ms boot, E2B/Modal are built for that. Box's sweet spot: tens of persistent, cheap, fully-isolated machines. EU-only; 100 active VMs soft cap (raising).

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Recipe

```bash
base="$(box new --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$base" -- bash -s < ./setup.sh                 # configure once
for i in $(seq 1 10); do
  fork="$(box fork "$base" --json | jq -r 'select(.event == "ready") | .id')"
  box ssh "$fork" -- "cd /project && ./run-task.sh $i" &
done
wait
```

Keep `setup.sh` and the start command idempotent so every fork bootstraps reliably.
