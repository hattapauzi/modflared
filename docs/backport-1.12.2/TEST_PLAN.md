# Modflared 1.12.2 Test Plan

## Build Validation

- Command: `./gradlew clean build`
- Expected: Build succeeds on Java 8-compatible toolchain and produces `build/libs/modflared-1.12.2-legacy.1.jar`.

## Launch Validation

- Command: `./gradlew runClient`
- Expected: Minecraft 1.12.2 opens with Forge, Modflared appears in the mod list, and the log contains `Modflared client setup complete`.

## Binary Validation

- Local binary present: Put a valid `cloudflared` on `PATH`, launch client, and confirm the log says `Using local cloudflared`.
- Local binary missing: Remove `cloudflared` from `PATH`, launch client, and confirm the mod downloads into `modflared/bin/`.
- Invalid download: Disconnect network during download and confirm a visible log error without a client crash.

## Connection Validation

| Case | Input | Expected |
| --- | --- | --- |
| Direct IP | A normal non-tunneled IP server | Connects directly; no cloudflared process starts. |
| Hostname TXT route | Hostname with `cloudflared-use-tunnel` or `cloudflared-route=<route>` TXT record | Starts cloudflared and connects to `127.0.0.1:<deterministic port>`. |
| Forced tunnel | Hostname in `modflared/forced_tunnels.json` | Starts cloudflared even without TXT records. |
| Saved server ping | Saved tunneled server in multiplayer list | Ping is routed through local cloudflared. |
| Saved normal ping | Saved non-tunneled server in multiplayer list | Ping is direct and does not start cloudflared. |

## Validation Run - 2026-04-30

| Case | Result | Notes |
| --- | --- | --- |
| `./gradlew clean build` | PASS | Build succeeded with no errors. JAR produced: `modflared-1.12.2-legacy.1.jar`. |
| `./gradlew runClient` | UNTESTED | Asset download fails with `NoRouteToHostException` in this environment (no internet to Minecraft asset servers). Mod entrypoint compiles and Mixin bootstrap is configured. |
| JAR contents | PASS | Contains all expected classes, `mcmod.info`, `modflared.mixins.json`, `modflared.refmap.json`, `forced_tunnels.json`, and `en_us.lang`. |
| Refmap alignment | PASS | Fixed mismatch: `modflared.mixins.json` referenced `modflared.refmap.json`, but JAR originally contained `mixin.refmap.json`. `build.gradle` now renames it at jar time. |
| Unit tests | PASS | 7/7 pass: `RunningTunnelAccessTest` (3), `CloudflaredDownloadTest` (3), `ForcedTunnelsJsonTest` (1). |
| Direct IP | UNTESTED | Requires client launch for end-to-end validation. |
| Hostname TXT route | UNTESTED | Requires client launch and network access for DNS resolution. |
| Forced tunnel | UNTESTED | Requires client launch. |
| Saved server ping | UNTESTED | Requires client launch. |
