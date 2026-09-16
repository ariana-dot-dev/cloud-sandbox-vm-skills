---
name: expose-localhost-tunnel
description: Tunnel/expose a localhost service to the internet from inside a cloud VM, get a stable public HTTPS URL (https://<sandbox>-<port>.on.boat.dev) with TLS and token-gating built in, no separate daemon or account like ngrok. The VM's dedicated IPv4 also allows raw TCP/UDP and bring-your-own domain. Use to share a preview, expose a webhook target, or open a port.
---

# Expose / tunnel localhost (Boat)

Use this when a service running in a Boat sandbox must be **reachable from outside**: sharing a dev preview, receiving webhooks, or demoing. Because the service already runs *inside* the VM, exposing it is one command: no `ngrok`-style sidecar, no extra account, no cert setup.

## Recipe

```bash
boat ssh "$sandbox_id" -- "cd /app && PORT=3000 npm start &"   # MUST bind 0.0.0.0
boat ssh "$sandbox_id" -- "host 3000 --title preview"
boat ssh "$sandbox_id" -- "host url 3000"                       # -> https://<sandbox>-3000.on.boat.dev?_token=...
boat ssh "$sandbox_id" -- "host list"                           # manage; host hide <port> to revoke
```

Boat terminates TLS for `on.boat.dev` and proxies to the sandbox. URLs are **token-gated by default** (`--public` drops the token, `--private` enforces it).

## Why Boat beats a tunnel tool

- No separate daemon/account: the route lives with the VM.
- TLS + auth handled; URL stable across the sandbox's life.
- **Dedicated IPv4 + full TCP/UDP**: non-HTTP servers and BYO domains work; most sandbox providers can't tunnel raw TCP/UDP at all.

## Prereqs

`curl -fsSL https://boat.dev/install | sh`, then `boat login "$BOAT_API_KEY" --json`.

## Gotchas

App **must bind `0.0.0.0`**. For a URL that outlives a long session, pair with `boat extend <id> --no-auto-stop`. EU-only region today.
