---
name: expose-localhost
description: Put a service running inside a cloud VM on a stable public HTTPS URL (https://<box>-<port>.on.ascii.dev) with TLS and token-gating built in — or use the VM's dedicated IPv4 for raw TCP/UDP and a bring-your-own domain. No separate tunnel daemon or account. Use to share a preview, expose a webhook target, or host a service for remote access.
---

# Expose a service publicly (Box)

Use this when something running in a Box needs to be **reachable from the outside**: sharing a dev preview, receiving webhooks, demoing an app, or letting an external service hit an endpoint.

Because the service already runs *inside* the VM, exposing it is one command — no `ngrok`-style sidecar, no extra account, no certificate setup.

## Two ways to expose

**1. Public HTTPS (most common).** Start the app on a port (bind `0.0.0.0`, not localhost) and run `host`:

```bash
box ssh "$box_id" -- "cd /app && PORT=3000 npm start &"   # must bind 0.0.0.0
box ssh "$box_id" -- "host 3000 --title 'preview'"
box ssh "$box_id" -- "host url 3000"                       # -> https://<box>-3000.on.ascii.dev?_token=...
```

Ascii terminates TLS for `on.ascii.dev` and proxies to the box. URLs are **token-gated by default** (`--public` to drop the token, `--private` to require it). `host list` / `host hide <port>` manage them.

**2. Raw TCP/UDP or your own domain.** Each Box has a **dedicated IPv4** with full TCP/UDP — so non-HTTP servers (databases, game servers, SSH, custom protocols) and BYO-domain setups work. Most sandbox providers can't do this at all.

## Why Box beats a tunnel tool here

- No separate daemon/account — the route lives with the VM.
- TLS + auth handled for you; URL is stable across the box's life.
- Full TCP/UDP + dedicated IPv4, not just HTTP tunneling.

## Prerequisites

```bash
curl -fsSL https://box.ascii.dev/install | sh
export BOX_API_KEY="bxk_..."        # from https://box.ascii.dev/box/dashboard
box login "$BOX_API_KEY" --json
```

## Gotchas

- The app **must bind `0.0.0.0`** — the public route comes from outside the VM's loopback.
- For a URL that must outlive a long session, pair with `box extend <id> --no-auto-stop`.
- EU-only region today.
