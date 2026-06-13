---
name: parallel-agents
description: Build an "agent factory" — configure one cloud VM once, then disk-level fork it into a fleet of identical, isolated machines so many agents run in parallel, each with its own filesystem, IPv4, and SSH. Flat per-second pricing makes a fleet ~10x cheaper than E2B/Modal (50 VMs ≈ $432/mo vs ~$4k). Use when you need to fan work out across many real machines instead of threads.
---

# Parallel agents / agent factory (Box)

Use this when a job should be **fanned out across many isolated environments**: running N agents on N tasks, sweeping a matrix of configs, or sharding a large workload — and each worker needs a *real* machine (its own disk, processes, network), not just a coroutine.

The Box move: configure **one** box (clone the repo, install deps, set secrets), then **fork** it. Each fork is an independent VM that starts from the configured snapshot, so you skip re-provisioning every machine.

## Why Box for this

- **Fork = a configured template.** One setup, N identical machines — disk-level fork, no re-cloning/re-installing per VM.
- **True isolation per agent.** Separate filesystem, dedicated IPv4, SSH — agents can't step on each other; they can even talk over the network.
- **Fleet economics.** Flat per-VM per-second pricing: 50 VMs × 240 h/mo ≈ **$432 on Box** vs ~$4,124 (E2B) / ~$7,160 (Modal) / ~$8,179 (Vercel Sandbox).

## Honest scoping

Box forks at the **disk** level, not memory — a forked VM does **not** keep the parent's running processes; re-run your start command after fork. If you need *running-process* fork or routinely need **1000+ concurrent** VMs with <500 ms boots, E2B or Modal are built for that scale. Box's sweet spot is **tens of persistent, cheap, fully-isolated machines**. Box is EU-only; 100 active VMs soft cap today (raising).

## Prerequisites

```bash
curl -fsSL https://box.ascii.dev/install | sh
export BOX_API_KEY="bxk_..."        # from https://box.ascii.dev/box/dashboard
box login "$BOX_API_KEY" --json
```

## Recipe

```bash
# 1. Configure a base box once
base="$(box new --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$base" -- bash -s < ./setup.sh      # clone repo, install deps, etc.

# 2. Fork it into a fleet
for i in $(seq 1 10); do
  fork="$(box fork "$base" --json | jq -r 'select(.event == "ready") | .id')"
  # forks don't inherit running processes — (re)start work explicitly
  box ssh "$fork" -- "cd /project && ./run-task.sh $i" &
done
wait

# 3. Tear the fleet down when done
box list --json | jq -r '.boxes[].id' | xargs -I{} box delete {} --json
```

Keep `setup.sh` and your start command **idempotent** so every fork bootstraps reliably.
