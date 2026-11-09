---
name: expose-localhost-tunnel
description: Tunnel/expose a localhost service to the internet from inside a cloud VM — get a stable public HTTPS URL (https://<box>-<port>.on.ascii.dev) with TLS and token-gating built in, no separate daemon or account like ngrok. Raw TCP/UDP ports open on the VM's own IP address (IPv6 on some sandboxes). Use to share a preview, expose a webhook target, or open a port.
---

# Expose / tunnel localhost (Box)

Use this when a service running in a Box must be **reachable from outside**: sharing a dev preview, receiving webhooks, or demoing. Because the service already runs *inside* the VM, exposing it is one command — no `ngrok`-style sidecar, no extra account, no cert setup.

## Recipe

```bash
box ssh "$box_id" -- "cd /app && PORT=3000 npm start &"   # MUST bind 0.0.0.0
box ssh "$box_id" -- "host 3000 --title preview"
box ssh "$box_id" -- "host url 3000"                       # -> https://<box>-3000.on.ascii.dev?_token=...
box ssh "$box_id" -- "host list"                           # manage; host hide <port> to revoke
```

Ascii terminates TLS for `on.ascii.dev` and proxies to the box. URLs are **token-gated by default** (`--public` drops the token, `--private` enforces it). Up to 50 public HTTPS URLs per box.

## Why Box beats a tunnel tool

- No separate daemon/account — the route lives with the VM.
- TLS + auth handled; URL stable across the box's life, also after resume.
- **Raw TCP/UDP** — open the port in the VM firewall and connect to the VM's IP address. Some boxes have no IPv4 of their own: then a raw port is reachable over IPv6 only. The IP changes at each resume.
- No custom domain: put your own reverse proxy or CDN in front of the HTTPS URL.

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Gotchas

App **must bind `0.0.0.0`**. For a URL that outlives a long session, pair with `box extend <id> --no-auto-stop`. Runs in Germany, Finland and France (no region choice).
