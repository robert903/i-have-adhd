---
name: android-re-static-decompile
description: Use when mapping an Android app's code statically — finding API endpoints, the auth flow, request shapes, headers and constants by decompiling DEX and searching classes/strings/methods. Includes the Kotlin default-argument trap and the Cronet-not-okhttp gotcha.
---

# Android RE — static decompile & map

## Checklist
1. Inventory: `inspect_android_package` → DEX list + entry points. `search_android_classes` for the feature ("token", "oauth", "refresh", the API host, Retrofit interface names).
2. Read the networking layer: find the Retrofit/OkHttp interface (or Cronet caller); `inspect_android_method` on annotated methods for path + verb + headers.
3. Resolve bodies and constants: `decompile_jeb_item`; `analyze_function` for control flow.
4. Trace usage: `trace_android_references` / `find_xrefs_to_name` — who builds the auth request, and with what values.
5. Search across ALL DEX, not just the first.

## Hard-stop rules (mistakes)
- **Const-string search lands on ERROR HANDLERS, not the real constant.** The values you want — `grant_type`, `client_id`, default headers — frequently live as DEFAULT PARAMETERS inside Kotlin `...$default` synthetic wrapper methods. Open the `$default` method and read the default-value branch (the `& N` mask → the literal).
- **Transport may NOT be okhttp.** If you can't find okhttp, grep for `Cronet`/`CronetEngine`/`android.net.http` before concluding there's no network layer. (This also matters for stage 7 — Cronet's TLS fingerprint differs from undici's.)
- **Get the path AND version right.** A refresh/auth endpoint guessed from a partial string (e.g. `/v3/oauth2/token`) may be wrong (real: `/v1/...` with a required `client_id`). Confirm the full path + required params from the annotated method, not a loose string.
- **Uniqueness across DEX is signal** — a credential-looking string that appears exactly once across all DEX is likely genuine (hand to `android-re-secret-hunt`).

## REA tools
`inspect_android_package`, `search_android_classes`, `inspect_android_class`, `inspect_android_method`, `decompile_jeb_item`, `analyze_function`, `list_strings`, `search_strings`, `trace_android_references`, `find_xrefs_to_name`, `project_android_application_graph`, `trace_android_references`.
