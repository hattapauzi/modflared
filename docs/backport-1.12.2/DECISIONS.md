# Modflared 1.12.2 Backport Decisions

## Locked Decisions

- Target Minecraft version: `1.12.2`.
- Target Forge version: `1.12.2-14.23.5.2860`.
- Target Java version: Java 8 bytecode.
- Mod side: client only.
- Supported OSes for first release: Windows and Linux.
- Tunneling model: `Minecraft client -> 127.0.0.1:<deterministic local port> -> local cloudflared process -> Cloudflare tunnel/access -> target Minecraft server`.
- Binary strategy: prefer valid local/system `cloudflared`, repair by downloading, fail only after both paths fail.
- Downloaded binary directory: `<minecraft-instance>/modflared/bin/`.
- Forced tunnel config path: `<minecraft-instance>/modflared/forced_tunnels.json`.
- Mixin is required for direct-connect and server-list ping routing.
- Publishing automation is out of scope for the first milestone.

## New Decisions

| Date | Decision | Reason | Impact |
| --- | --- | --- | --- |
| 2026-04-30 | Use ForgeGradle 2.3 and Forge `1.12.2-14.23.5.2860`. | The spec locks this Forge build. | Build files must use legacy Gradle patterns compatible with ForgeGradle 2.3. |
| 2026-04-30 | Pin Gradle wrapper to `4.10.3`. | ForgeGradle 2.3 is incompatible with Gradle 9.x used by the modern branch. | `gradle-wrapper.properties` must reference a 4.x distribution. |
| 2026-04-30 | Use logging-only setup failure reporting for the first buildable 1.12.2 milestone. | Modern toast APIs do not exist on 1.12.2 and the spec requires basic visible logs first. | GUI notification parity can be revisited after connection routing works. |
