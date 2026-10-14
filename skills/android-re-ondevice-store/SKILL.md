---
name: android-re-ondevice-store
description: Use when extracting an app's live on-device data — token stores, shared-prefs, SQLite databases — over adb for a reverse-engineering project. Read-only. Includes the harvest-don't-refresh pattern for self-refreshing tokens.
---

# Android RE — on-device store extraction

Often the FASTEST path to a working client: skip replicating auth and just read the token the app already holds.

## Checklist
1. Find the store: `list_adb_directory` under `/data/data/<pkg>/` → `files/`, `shared_prefs/`, `databases/`. Root `cat` or `pull_adb_file`.
2. Identify the format FIRST, then parse:
   - plain SharedPrefs XML;
   - SQLite `.db` (open read-only);
   - **Harmony `prefs.data`** = JSON `{metaData:{name}, data:[{key,value,type}]}` → map each `key`→`value` (e.g. `ets_access_token[]`, `ets_refresh_token[]`, `ets_expires_at[]` ms, `ets_token_lifetime[]`).
3. Extract only the fields you need (tokens, expiry, lifetime, device id).

## Hard-stop rules
- **READ-ONLY.** Snapshot the file; NEVER write back to the live store. A corrupted token write logs the real account out (silently, until next launch).
- **Harvest, don't replicate.** If the app self-refreshes a long-lived token (~24h) forever, COPY its live token on a schedule (every ~half-life, e.g. 12h) instead of reimplementing the undocumented refresh protocol. Mirror it into your client's secrets with an atomic write at 0600; make your long-running client re-read the file (a reload hook) so it picks up each refresh.
- **Run the harvester on an always-on host** that can adb-reach the phone (authorize that host's adb key — `android-re-device-access`). The phone and an always-on server both on a tailnet is ideal.
- Keep the harvest a pure local adb copy (no upstream request) unless you deliberately want a verify-against-API step.

## REA tools
`inspect_adb_package`, `list_adb_directory`, `pull_adb_file`.
