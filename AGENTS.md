# Repository Guidelines

## Project Structure & Module Organization

Modflared is a Java 21 NeoForge client mod. Core source code lives in `src/main/java/dev/httxrafa/modflared`, grouped by responsibility: `binary` handles cloudflared binaries, `github` handles release lookup, `tunnel` manages tunnel lifecycle, `mixin` contains Minecraft mixins, and `interfaces/mixin` contains mixin accessors. Runtime resources are in `src/main/resources`, including `modflared.mixins.json`, `forced_tunnels.json`, and `assets/modflared/lang/en_us.json`. Mod metadata templates live in `src/main/templates` and are expanded by Gradle into generated resources. README images are stored under `.github/images`.

## Build, Test, and Development Commands

Use the Gradle wrapper; do not assume a system Gradle version.

- `./gradlew build` compiles Java, processes resources, and builds the mod jar.
- `./gradlew runClient` launches a local Minecraft client for manual testing.
- `./gradlew runServer` launches a local server with `--nogui`.
- `./gradlew runGameTestServer` runs NeoForge game tests if any are registered.
- `./gradlew modrinth` and `./gradlew publishCurseForge` publish releases; they require `MODRINTH_TOKEN` or `CURSEFORGE_TOKEN`.

## Coding Style & Naming Conventions

Use Java 21, UTF-8, and the package root `dev.httxrafa.modflared`. Follow existing formatting: 4-space indentation, braces on the same line, uppercase enum constants, `PascalCase` classes, `camelCase` methods and fields, and descriptive package names by feature. Keep mixin classes in `mixin` or `mixin/client`, and keep accessor interfaces prefixed with `I` in `interfaces/mixin`.

## Testing Guidelines

There is currently no `src/test` tree or separate unit-test framework configured. Validate changes with `./gradlew build` and, for behavior touching Minecraft networking, connection screens, tunnel startup, or binary downloads, run `./gradlew runClient` and test a local or configured Cloudflare tunnel path. Add GameTests only when they can run headlessly through NeoForge.

## Commit & Pull Request Guidelines

Recent history uses short imperative subjects and lightweight Conventional Commit prefixes such as `feat:` and `fix:`. Use examples like `feat: Update to NeoForge 1.21.11` or `fix: Publish workflow`. Keep PRs focused, describe user-visible behavior changes, list validation commands run, link issues when applicable, and include screenshots only for UI or README image changes.

## Security & Configuration Tips

Do not commit publish tokens, Cloudflare credentials, tunnel IDs, or local generated run data. Keep release configuration in `gradle.properties`, but pass secrets through environment variables or GitHub Actions secrets.
