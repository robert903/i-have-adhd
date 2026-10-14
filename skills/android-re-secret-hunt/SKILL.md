---
name: android-re-secret-hunt
description: Use when locating and CONFIRMING credentials, API keys, or client secrets inside a decompiled Android app. Encodes the "don't call a real credential a decoy" mistake — confirm genuineness by cross-DEX uniqueness plus a live API test, and classify public-client vs user secrets.
---

# Android RE — find & confirm secrets

## Checklist
1. Search: `search_strings` for Basic/Bearer prefixes, base64 blobs, `client_id`/`client_secret`/`api_key`/`apikey`, and 32/40/64-char hex.
2. Decode: base64-decode `Basic` headers to `user:secret`; note the structure.
3. Locate the use site: `trace_android_references` to see which request actually sends the value at runtime.
4. CONFIRM before trusting OR dismissing (see rules).

## Hard-stop rules (the "decoy" mistake)
- **Never label a credential a "decoy" or "fake" without proof.** We wrongly dismissed a genuine client credential and dead-ended the whole refresh path. Prove genuineness two ways: (a) **cross-DEX uniqueness** — the value appears exactly once across all DEX; (b) a **live API test** — it actually authenticates. A decoy is only proven when it *fails a real request AND a different value is used at runtime*.
- **Classify the secret.** A PUBLIC CLIENT secret (shipped in every install — e.g. an OAuth `client_id:client_secret` for a public client) is genuine and safe to reuse for your OWN account. A USER secret (password, refresh/access token) is per-account and must come from the live store (stage 5), not the APK.
- **Cross-check community clients, but verify.** A value matching a known open-source client for the same app is a strong hint — still run the live test.

## Output discipline
Record: the value (written to a 0600 secrets file, NEVER printed to the terminal or a commit), where it's used, and how you confirmed it. Never commit it (gitignore the secrets dir). Never send the user's email to a service.
