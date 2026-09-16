---
name: public-url-hosting
description: Host a service on a stable public HTTPS URL straight from a cloud VM, start your app, run one command, get https://<sandbox>-<port>.on.boat.dev with managed TLS and token-gating. Supports bring-your-own domain and raw TCP/UDP via a dedicated IPv4. Use to host a web app, API, demo, or webhook endpoint without DNS or certificate setup.
---

# Public URL hosting (Boat)

Use this to **put a running service on the public internet** from inside a Boat sandbox: a web app, an API, a demo, or a long-lived endpoint. Boat handles TLS, routing, and auth: no DNS, no certificates, no separate platform.

## Recipe

```bash
sandbox_id="$(boat new --no-auto-stop --json | jq -r 'select(.event == "ready") | .id')"
boat ssh "$sandbox_id" -- bash -s < ./deploy.sh                 # build + start app on 0.0.0.0:8080
boat ssh "$sandbox_id" -- "host 8080 --public --title 'my app'"
boat ssh "$sandbox_id" -- "host url 8080"                        # https://<sandbox>-8080.on.boat.dev
```

`host` URLs are token-gated by default; `--public` makes them open. A dedicated IPv4 + full TCP/UDP means you can also point your own domain or host non-HTTP services.

## Why Boat

- **Zero infra**: TLS + subdomain + auth managed for you.
- **Stays up**: `--no-auto-stop` keeps the host alive 24-7; persists across resume.
- **Cheap**: per-second billing, ~10× cheaper than typical sandbox/host providers.

## Prereqs

`curl -fsSL https://boat.dev/install | sh`, then `boat login "$BOAT_API_KEY" --json`.

## Limits

EU-only; 100 active VMs soft cap. For multi-region CDN-grade hosting, use a dedicated host; Boat is best for single-VM services, previews, and demos.
