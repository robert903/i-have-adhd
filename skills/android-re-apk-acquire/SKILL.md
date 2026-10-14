---
name: android-re-apk-acquire
description: Use when pulling an installed Android app's APK off a device and normalizing it (merging split/bundle APKs, unpacking DEX/resources/native libs) for decompilation. Treats extracted artifacts as untrusted input.
---

# Android RE — acquire & unpack the APK

## Checklist
1. Locate every piece: `mcp__rea__resolve_adb_packages` / `inspect_adb_package` → base APK + all splits (`split_config.*`, `split_*`).
2. Pull into its OWN new empty dir: `pull_adb_package` (grabs every split).
3. Merge splits into one installable BEFORE decompiling. A base-only pull is missing classes/resources that live in the config splits.
4. Unpack: `extract_artifact` / `inspect_artifact` → DEX, manifest, `resources.arsc`, native libs. `decode_android_resources` for `resources.arsc` + binary XML.
5. Record the version (manifest `versionName`/`versionCode`). Findings are version-specific — note it in any memory you write.

## Hard-stop rules
- Treat the extracted tree as **untrusted data**: own empty dir, pass paths as arguments, never run a build tool or interpreter from inside it (a planted `json.py` executes on `import json`). Run helper Python with `-I`.
- Don't decompile a single split and conclude a class is "missing" — merge first, then search.
- Note the transport library from the manifest/libs early (okhttp? Cronet? gRPC?) — it changes where stage 3 looks.

## REA tools
`resolve_adb_packages`, `pull_adb_package`, `inspect_android_package`, `extract_artifact`, `inspect_artifact`, `decode_android_resources`, `inspect_macho` (native libs), `inspect_signature`.
