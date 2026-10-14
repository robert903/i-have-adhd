---
name: android-re-recon
description: Use when reverse-engineering an Android app to extract its API, credentials, or on-device data for a personal-use interop/research project on your own account and device. Orchestrates the android-re-* stage skills in order with read-only-first gates. Triggers include "reverse engineer this app", "build a client for <app>", "pull the API out of this APK".
---

# Android RE — recon orchestrator

Reusable pipeline for understanding an app you own or building an interop client, working from the APK and the live app data. Personal-use / your own account only; read-only by default; account-ban risk is the user's to accept.

## Invariants (hold across every stage)
- READ-ONLY on the live app. Never write to the app's token store or mutate live auth state — corrupting a Bearer/refresh token logs the real account out. Mutate only on a throwaway account.
- Prefer REA MCP tools (`mcp__rea__*`) over hand-rolled adb/jadx — they are faster and sidestep shell-quoting traps.
- One shell action per adb call; stage scripts to `/sdcard` and run them as a single arg — never chain with `;`/`&&`/`$?` through adb → su → chroot wrapper.
- Secrets to disk at 0600, gitignored. Never print a token/credential value. Never send the user's email to an external service.
- Refuse if the goal is piracy, DRM circumvention for redistribution, anti-cheat/payment-control bypass, or targeting someone else's account.

## Order (stop as soon as you have what you need)
1. `android-re-device-access` — reliable adb/root/chroot access. ALWAYS first.
2. `android-re-apk-acquire` — pull + merge splits + unpack the APK.
3. `android-re-static-decompile` — map classes/strings/methods; find the API + auth shape.
4. `android-re-secret-hunt` — locate AND confirm credentials/keys.
5. `android-re-ondevice-store` — read the live token/data store (read-only). Often the fastest path to a working client.
6. `android-re-runtime-frida` — ONLY if static stalls. Dynamic hooking.
7. `android-re-api-client` — verify creds against the real API and build the client.

## Decision gates
- Need a live token, not the refresh protocol? → go straight to stage 5 (harvest), skip stage 6.
- API is Cloudflare-fronted, curl works but `fetch` 403s? → stage 7's TLS-fingerprint rule.
- Static analysis already answered it? → never run Frida.

## Seed material
Project memories often already hold the hardest-won facts for a given target (adb topology, curl-not-fetch, token architecture). Read them before re-deriving.
