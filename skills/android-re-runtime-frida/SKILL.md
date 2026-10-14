---
name: android-re-runtime-frida
description: Use when static analysis stalls and you need dynamic instrumentation of an Android app — hooking methods, observing calls, dumping runtime values with Frida. Includes the Frida 17 Java-bridge change and the never-mutate-live-auth rule.
---

# Android RE — runtime instrumentation (last resort)

Only reach here if `android-re-static-decompile` and `android-re-ondevice-store` did NOT answer the question. Try REA's read-only runtime observers FIRST (`observe_native_calls`, `observe_javascript_runtime`) before hand-writing a Frida agent.

## Checklist
1. `frida-server` matching the device arch, running as root; Frida tools on the host (a `uv tool` Python env for the Python API).
2. Write the agent, compile it, attach, drive the session from the Python API.
3. Hook READ-ONLY first: log args/returns of the auth builder / request signer; don't change anything.

## Hard-stop rules (mistakes)
- **Frida 17 REMOVED the global `Java` object.** `Java.perform(...)` throws "Java is not defined". FIX: `frida-compile` an agent that imports `frida-java-bridge` (a single ESM module that re-exposes `Java`), and drive the session from the Python API — not an inline script string that assumes the old global.
- **NEVER mutate live auth state.** We corrupted a Bearer JWT via a hook and logged the real user out. Observe and log only. If you must forge/mutate a request to test, do it on a THROWAWAY account — never the primary.
- Treat any write to auth state as destructive: logout is usually silent until the next app launch, so you won't get a warning.

## REA tools (prefer these — read-only by construction)
`observe_native_calls`, `observe_javascript_runtime`, `observe_native_ui`, `trace_native_values`, `correlate_static_and_runtime`, `resolve_native_call_targets`.
