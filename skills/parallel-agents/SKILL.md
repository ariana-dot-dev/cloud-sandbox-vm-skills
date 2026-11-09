---
name: parallel-agents
description: Build an "agent factory" — configure one cloud VM once, then disk-level fork it into a fleet of identical isolated machines so many agents run in parallel, each with its own filesystem, IP address, and SSH. 50 VMs × 240 h/mo ≈ $432 on Box vs ~$4,124 (E2B) at list price. Use to fan work across many real machines instead of threads.
---

# Parallel agents / agent factory (Box)

Use this when work should be **fanned out across many isolated environments**: N agents on N tasks, a config sweep, or sharding a workload — and each worker needs a *real* machine, not a coroutine. Box move: configure **one** box (clone, install, secrets), then **fork** it; each fork is an independent VM from that snapshot, so you skip re-provisioning every machine.

## Why Box

- **Fork = a configured template** — one setup, N identical machines (disk-level, no re-clone per VM). A fork of a running box uses its last 1-minute snapshot.
- **True isolation per agent** — separate VM and filesystem, own public IPv6, SSH; agents can talk over the network.
- **Fleet price** — 50 VMs × 240 h/mo ≈ **$432 (Box)** vs ~$4,124 (E2B, incl. $150 Pro plan) / ~$5,711 (Modal, 1 core = 2 vCPU), at list price for VMs that run the full hours. Providers that bill only busy CPU can cost less for idle agents. Each fork counts as a start: the $20 plan includes 200 a day, extra starts cost $0.03-$0.07.

## Honest scoping

Box forks at the **disk** level, not memory — a fork does **not** keep the parent's running processes; re-run your start command after fork. Modal and E2B also fork disk state; for *running-process* restore or sub-second boot, E2B/Modal are built for that. Concurrency per plan: 100 at once on $20, 300 on $100, 1,000 on $500, 2,000 on $2,000 (https://docs.boat.dev/limits). Runs in Germany, Finland and France (no region choice).

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
