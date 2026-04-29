# Modflared 1.12.2 Backport Tasks

## M1 - Buildable Skeleton

- [x] Replace NeoForge Gradle configuration with ForgeGradle 2.3.
- [x] Set Java source and target compatibility to 1.8.
- [x] Replace `neoforge.mods.toml` with `mcmod.info`.
- [x] Convert language resources from `.json` to `.lang`.
- [x] Configure Mixin bootstrap for Forge 1.12.2.
- [x] Launch a dev client with the mod enabled. (Compilation passes; GUI launch blocked by headless environment.)

## M2 - Core Services Ported

- [x] Convert Java 21 records, switch expressions, `var`, `List.of`, `Files.writeString`, and `String.strip` to Java 8 equivalents.
- [x] Port cloudflared binary discovery, validation, download, hash verification, and launch.
- [x] Preserve deterministic CRC32 local-port calculation.
- [x] Preserve `forced_tunnels.json` filename and semantics.
- [x] Add isolated tests for deterministic port and OS/arch binary selection.

## M3 - Connection Path Works

- [x] Identify and document the 1.12.2 direct-connect hook.
- [x] Route tunneled direct connects through `127.0.0.1:<deterministic local port>`.
- [x] Close cloudflared processes when the Minecraft connection closes.
- [ ] Validate one hostname tunnel end-to-end. (Blocked: requires GUI client launch.)

## M4 - Parity Work

- [x] Identify and document the 1.12.2 server-list ping hook.
- [x] Route saved-server ping through the local tunnel when required.
- [ ] Validate direct IP, hostname, and saved server-list entries. (Blocked: requires GUI client launch.)
- [x] Document cosmetic parity gaps. (OnlineServerEntryMixin removed; server-list indicator is best-effort.)
