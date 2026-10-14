---
name: android-re-device-access
description: Use when establishing or troubleshooting adb, root, or proot/chroot access to an Android device for reverse engineering or on-device file extraction. Covers adb-server-vs-device ports, adb key authorization, Tailscale access, and the shell-quoting traps that cause phantom "not found" / "No such file" errors.
---

# Android RE — device access (do this first)

## Checklist
1. Map the topology BEFORE anything, and record it. Per device: adb **server** port (host side, e.g. a non-default `ANDROID_ADB_SERVER_PORT=5137`) vs device **tcp** port (usually 5555); Tailscale/LAN IP; rooted? Two devices on different server ports means exporting the right port per command.
2. Connect: `adb connect <ip>:5555`, confirm with `adb devices` (or `mcp__rea__list_adb_devices` / `inspect_adb_device`). TCP adb drops — reconnect at the start of a batch.
3. Root: via Magisk/`su`. Confirm `id` → uid=0.
4. Authorize a NEW host's adb key: append its `~/.android/adbkey.pub` to the device's `/data/misc/adb/adb_keys`; keep owner `system:shell`, mode `0640`. adbd reads it on the next connection — no restart needed. Generate the host key first with `adb start-server` (creates `~/.android/adbkey[.pub]`).

## Hard-stop rules (mistakes that cost real time)
- **Never chain commands with `;`, `&&`, or `$?` through adb shell → su → chroot wrapper.** The stacked shells re-parse and silently split the args — you get `apt-get: not found` (it ran on Android, not the chroot) or a `grep` that "can't find" a file the previous `grep` just read. FIX: write the whole command to a `.sh`, push to `/sdcard`, run `wrapper "bash /sdcard/x.sh"` as ONE clean arg.
- **One action per adb call** when verifying. Chained verifies produce phantom failures that look like real ones.
- **proot/chroot bridge:** `/sdcard` (bind-mounted into the chroot) is the only reliable way in/out. GNU `tar xzf` fails with ENOSYS on the proot overlay — extract with Python `tarfile` (open/write, strip the leading path component), not `tar`.
- **Self-deletion:** bash reads a script incrementally; a script that `rm`s itself (e.g. via a `*.sh` glob in its own body) dies mid-read. Don't delete the running script from inside it.

## REA tools
`list_adb_devices`, `inspect_adb_device`, `inspect_adb_package`, `list_adb_directory`, `list_adb_processes`, `pull_adb_file`, `push_adb_file`, `read_adb_logcat`.
