---
name: public-url-hosting
description: Host a service on a stable public HTTPS URL straight from a cloud VM — start your app, run one command, get https://<box>-<port>.on.ascii.dev with managed TLS and token-gating. Supports bring-your-own domain and raw TCP/UDP via a dedicated IPv4. Use to host a web app, API, demo, or webhook endpoint without DNS or certificate setup.
---

# Public URL hosting (Box)

Use this to **put a running service on the public internet** from inside a Box: a web app, an API, a demo, or a long-lived endpoint. Box handles TLS, routing, and auth — no DNS, no certificates, no separate platform.

## Recipe

```bash
box_id="$(box new --no-auto-stop --json | jq -r 'select(.event == "ready") | .id')"
box ssh "$box_id" -- bash -s < ./deploy.sh                 # build + start app on 0.0.0.0:8080
box ssh "$box_id" -- "host 8080 --public --title 'my app'"
box ssh "$box_id" -- "host url 8080"                        # https://<box>-8080.on.ascii.dev
```

`host` URLs are token-gated by default; `--public` makes them open. A dedicated IPv4 + full TCP/UDP means you can also point your own domain or host non-HTTP services.

## Why Box

- **Zero infra** — TLS + subdomain + auth managed for you.
- **Stays up** — `--no-auto-stop` keeps the host alive 24-7; persists across resume.
- **Cheap** — per-second billing, ~10× cheaper than typical sandbox/host providers.

## Prereqs

`curl -fsSL https://box.ascii.dev/install | sh`, then `box login "$BOX_API_KEY" --json`.

## Limits

EU-only; 100 active VMs soft cap. For multi-region CDN-grade hosting, use a dedicated host; Box is best for single-VM services, previews, and demos.
