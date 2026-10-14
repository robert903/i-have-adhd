---
name: android-re-api-client
description: Use when verifying recovered/harvested credentials against a real app API and building an interop client — especially when the API is behind Cloudflare and Node fetch is blocked by TLS/JA3 fingerprinting. Covers device-identity headers, safe token handling, and common response-value quirks.
---

# Android RE — verify & build the interop client

## Checklist
1. Reproduce ONE authenticated request with `curl`: the token PLUS the device-identity headers the app sends (`X-Device-Id`/device id, `User-Agent`, `Accept`, any app-version header). Expect 200.
2. Replicate device identity exactly — a token alone often isn't enough; the backend checks a device-id header against the token. Pull device id / UA from the app (static or on-device store).
3. Build the client as a thin proxy around curl: clean local API out, credentials in; frontend holds NO credentials.

## Hard-stop rules (the undici 403)
- **Cloudflare-fronted APIs fingerprint the TLS/JA3 handshake.** Node's undici (global `fetch`) gets a 403 even with correct headers + token; `curl`'s fingerprint passes. USE `curl` (or a fingerprint-matching client) for the upstream — NEVER Node `fetch`.
- **Token to curl via a 0600 `-K` config file, NEVER on argv** (argv is world-readable in `/proc`). `rm` the config in a `finally`.
- Bind the proxy to `127.0.0.1` by default; `0.0.0.0` only behind a trusted reverse proxy or on a private tailnet.
- Read-only client unless the user has explicitly accepted write/ban risk.

## Response-value quirks to normalize at the proxy edge
- Booleans arrive as `"1"`/`"0"` strings.
- Sentinel numbers (e.g. `speed: -1` = no GPS fix).
- Empty arrays can mean "free tier / feature off", not "no data".

## Deploy pattern that worked
Single container/process: the proxy serves BOTH the API and the built SPA (gate static serving behind a `WEB_DIR` env var so dev/tests are unaffected). Host on an always-on box that can also run the token harvester (`android-re-ondevice-store`). Request-driven, so it sleeps until opened.
