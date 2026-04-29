# Modflared 1.12.2 Backport Tech Spec

Status: Draft v1 candidate
Audience: Human maintainers and LLM coding agents
Branch intent: backport current `neoforge/1.21.11` functionality to Minecraft 1.12.2 on Java 8, using the existing `forge/1.12.2` branch as the working branch for a clean eventual PR.

This document is the source of truth for the 1.12.2 backport effort. It defines scope, constraints, compatibility assumptions, architectural boundaries, acceptance criteria, and the implementation plan for agentic programming workflows.

The backport is not treated as a mechanical version downgrade. It is treated as a targeted re-platforming from a Java 21 NeoForge client mod to a Java 8 Forge 1.12.2 client mod while preserving the core user-visible behavior of Modflared.

## 1. Inputs and precedent

This spec is based on:

* The current repository guidelines and structure of Modflared.
* The prior backport precedent in commit `2c5e44a` ("Backported modflared to 1.16.5").
* The open user request for 1.12.2 support in issue `#37`.
* Maintainer boundary decisions captured in this conversation.

## 2. Product goal

Deliver a Forge 1.12.2 client mod that preserves the core behavior of Modflared:

1. Detect whether a server should be tunneled through Cloudflare.
2. Start and manage a local `cloudflared` process when needed.
3. Route Minecraft client connection flow through the tunnel.
4. Preserve existing user-facing configuration concepts where practical, especially `forced_tunnels.json`.
5. Preserve server-list ping behavior as a required compatibility target.
6. Remain client-side only unless a later decision expands scope.

## 3. Required traffic model

The backport is expected to preserve this end-to-end flow when tunneling is active:

`Minecraft client -> 127.0.0.1:<deterministic local port> -> local cloudflared process -> Cloudflare tunnel/access -> target Minecraft server`

This flow is a hard architectural constraint for v1.

## 4. Non-goals

Unless explicitly approved later, this backport does not aim to:

* Preserve exact internal package layout from the NeoForge 1.21.11 branch.
* Share binary compatibility with newer loader branches.
* Add support for Fabric 1.12.2.
* Redesign feature behavior beyond what is necessary for 1.12.2 compatibility.
* Introduce unrelated feature work while backporting.
* Optimize for cross-version multi-loader abstraction in the first delivery.
* Add release publishing automation in the first milestone.

## 5. Source branch and target platform

### Baseline source

* Functional baseline: `neoforge/1.21.11`
* Historical precedent: prior 1.16.5 backport (`2c5e44a`)
* Working branch for the 1.12.2 effort: `forge/1.12.2`

### Target runtime

* Minecraft: 1.12.2
* Mod loader: Forge 1.12.2
* Java: 8
* Side: client
* Supported desktop OSes for first release: Windows and Linux

## 6. Primary constraints

1. Java language ceiling is Java 8.
2. Loader and runtime APIs differ substantially between NeoForge 1.21.11 and Forge 1.12.2.
3. Networking internals and multiplayer UI classes are different enough that mixins and injection points must be rediscovered, not copied blindly.
4. Logging stack, metadata format, resources, and packaging will change.
5. If current dependencies require Java greater than 8 or Minecraft greater than 1.12.2, they must be downgraded, replaced, shaded, or removed.
6. Dependency replacement is allowed only as a last resort and must be documented with the reason, affected behavior, and any known divergence.
7. The mod jar must not bundle `cloudflared`.

## 7. Guiding engineering principles

1. Preserve behavior first, implementation shape second.
2. Minimize simultaneous change. Port one subsystem at a time.
3. Prefer explicit compatibility notes over hidden assumptions.
4. Keep the first deliverable narrow and working.
5. Record every intentional deviation from the 1.21.11 branch in this spec or `DECISIONS.md`.
6. Any LLM agent making code changes must update this spec when scope, assumptions, or decisions change.
7. Optimize this spec for human maintainers first.

## 8. Functional scope for v1

### In scope

* Client mod bootstrap on Forge 1.12.2
* Cloudflared runtime strategy: local-first, download-if-needed
* Cloudflared binary selection, validation, download, and launch for supported operating systems
* Tunnel lifecycle management
* Domain and route detection logic needed for automatic tunnel usage
* Forced tunnel configuration via `forced_tunnels.json`
* Connection-flow integration required to route client connections through the local tunnel
* Server-list ping compatibility matching the modern branch as closely as practical
* Basic user-visible error reporting and logging

### Out of scope until explicitly approved

* Feature additions unrelated to backporting
* Automatic migration tooling for configs from modern branches
* Release publishing automation
* Multi-loader abstractions intended to be shared with newer branches

## 9. Architecture workstreams

### Workstream A: Build system and metadata

Goal: create a clean Forge 1.12.2 / Java 8 build that can compile and run.

Expected tasks:

* Replace NeoForge-specific build configuration with Forge 1.12.2 Gradle setup.
* Set `sourceCompatibility` and `targetCompatibility` to Java 8.
* Replace NeoForge metadata and resources with Forge 1.12.2 equivalents.
* Rework mixin bootstrap for a Forge 1.12.2 environment.
* Restore or replace launch and run configurations for client testing.

Outputs:

* Compiling Forge 1.12.2 project
* Launchable dev client

### Workstream B: Runtime bootstrap and platform integration

Goal: port mod entrypoints and lifecycle hooks.

Expected tasks:

* Introduce the Forge 1.12.2 main mod entrypoint.
* Re-map initialization stages from NeoForge lifecycle to Forge lifecycle.
* Re-home platform-specific initialization that currently assumes NeoForge.

Outputs:

* Mod loads in a Forge client without crashing during startup

### Workstream C: Common domain logic portability

Goal: salvage as much version-agnostic logic as practical.

Candidate modules for direct or near-direct reuse:

* cloudflared binary lookup, validation, and download policy
* tunnel process lifecycle concepts
* config file parsing and persistence semantics
* DNS and route resolution logic that does not depend on Minecraft internals

Porting rule:

* If a class is pure Java and only needs Java 8 syntax downgrades, port it with minimal semantic changes.
* If a class touches Minecraft classes or loader APIs, treat it as version-specific.

### Workstream D: Connection interception and networking

Goal: reproduce the required connection rerouting and server-list behavior on 1.12.2.

This is the highest-risk workstream.

Injection policy:

* Use Mixin-based bytecode injection into Minecraft client networking and multiplayer UI.
* The mixin config is required.
* Do not optimize for loader-agnostic abstraction or common-code shims in this branch.

Expected tasks:

* Identify 1.12.2 classes responsible for multiplayer direct-connect flow, server-list ping flow, and client connection state transitions.
* Recreate the tunnel decision point and local reroute logic around the deterministic local port model.
* Document the direct-connect hook.
* Document the server-list ping hook.
* Document the tunnel process lifecycle touchpoints.
* Document the config load path for `forced_tunnels.json`.
* Validate behavior for:

  * direct connection to tunneled hostname
  * connection through forced tunnels entry
  * normal non-tunneled server connection
  * server-list ping behavior

Outputs:

* A documented injection map for the high-risk touchpoints listed above
* One working connection path end-to-end for milestone completion
* Clear notes on any remaining parity gaps versus `neoforge/1.21.11`

### Workstream E: Logging and diagnostics

Goal: ensure debugging is possible on the older stack.

Expected tasks:

* Standardize logging API for Forge 1.12.2.
* Preserve meaningful operational logs around binary download, binary validation, tunnel startup, DNS resolution, forced-tunnels parsing, and connection handoff.
* Document expected log markers for validation.

## 10. Known precedent from the 1.16.5 backport

The earlier backport indicates the following categories of change are likely required again:

* Loader branch change from NeoForge or Fabric-oriented layout to Forge-oriented layout
* Logging API adjustments
* New or altered mixins for multiplayer ping and connect behavior
* Conditional handling for API differences in server address validation and networking flow
* Build and branch logic updates

The 1.12.2 backport should assume more severe API drift than 1.16.5 and should not assume patch-level similarity.

## 11. Proposed repository shape for the backport branch

Preferred initial shape:

* Keep the repository single-purpose for the backport branch.
* Avoid forcing a shared multi-loader architecture during the first pass.
* Use a simple package structure optimized for Forge 1.12.2 maintainability.

Proposed top-level docs:

* `docs/backport-1.12.2/TECH_SPEC.md` — canonical spec
* `docs/backport-1.12.2/DECISIONS.md` — ADR-style decision log
* `docs/backport-1.12.2/TASKS.md` — actionable implementation backlog
* `docs/backport-1.12.2/TEST_PLAN.md` — manual validation matrix
* `docs/backport-1.12.2/API_DIFF_NOTES.md` — class, method, and injection mapping notes

## 12. LLM agent operating contract

Any coding agent working from this spec must follow these rules:

1. Read this spec before making edits.
2. Prefer small, reviewable patches.
3. Do not mix build migration, runtime bootstrap, and connection-hook rewrites in a single patch unless unavoidable.
4. Update `DECISIONS.md` when making irreversible compatibility choices.
5. Update `TASKS.md` when completing or splitting work.
6. Do not claim feature parity unless validated in `TEST_PLAN.md`.
7. Do not upgrade the target off Java 8 / Forge 1.12.2 without explicit approval.
8. Record blocked items and unknowns rather than guessing silently.
9. When replacing a dependency, document the reason, affected behavior, and known divergence.
10. Treat direct-connect hook, server-list ping hook, tunnel lifecycle, and `forced_tunnels.json` load path as high-priority mapping targets.

## 13. Acceptance criteria for milestone completion

The first approved milestone is considered successful when all of the following are true:

1. The mod compiles on Java 8 for Forge 1.12.2.
2. The client launches with the mod enabled.
3. The mod can locate a valid local `cloudflared` binary or repair the environment by downloading one at runtime when needed.
4. Connecting to a configured tunneled server succeeds through the local tunnel path.
5. Failures are visible in logs with enough detail to debug.

These milestone criteria are narrower than final v1 parity expectations. Full practical parity, including server-list ping behavior and stable config behavior where practical, remains part of the broader target scope.

## 14. Risks

### Technical risks

* Forge 1.12.2 connection flow may require a substantially different interception design.
* Older mixin/bootstrap support may be brittle or incompatible with the intended toolchain.
* Modern libraries used by the current branch may not support Java 8.
* Cloudflared binary support and packaging assumptions may differ on old environments.
* DNS and networking behavior may differ enough to require behavior changes.

### Project risks

* Scope creep toward full historical compatibility.
* Hidden parity expectations from users.
* Backport effort may stall without a strict milestone order.

## 15. Milestone plan

### M0 — Spec lock

* Confirm exact Forge build, supported OSes, deterministic port expectations, and binary placement strategy.
* Locked target Forge build: `1.12.2-14.23.5.2860`.

### M1 — Buildable skeleton

* Forge 1.12.2 project compiles on Java 8.
* Basic mod entrypoint loads.
* Mixin bootstrap is present.

### M2 — Core services ported

* Binary management, config parsing, and tunnel lifecycle compile and run in isolation.

### M3 — Connection path works

* One tunneled connection path works end-to-end through the required local tunnel model.

### M4 — Parity work

* Server-list ping path is implemented.
* Remaining parity gaps are documented.

## 16. Locked boundary decisions

The following decisions are locked unless explicitly revised later:

1. Target runtime is Minecraft 1.12.2 on Java 8.
2. The working line should use the existing `forge/1.12.2` branch so the eventual PR stays clean.
3. Target Forge build is `1.12.2-14.23.5.2860`.
4. Full practical parity with the modern branch is the target scope.
5. The required tunneling model is:
   `Minecraft client -> 127.0.0.1:<deterministic local port> -> local cloudflared process -> Cloudflare tunnel/access -> target Minecraft server`
6. Server-list ping behavior is required, not optional.
7. If some modern behavior is too invasive to port cleanly, the default policy is to drop it and document the gap.
8. Injection policy is Mixin-first. The mixin config is required.
9. First supported desktop OSes are Windows and Linux.
10. Cloudflared distribution strategy is local-first, download-if-needed. The mod jar must not bundle `cloudflared`.
11. Recovery chain for `cloudflared` validity is: prefer local/system when valid, repair by downloading when invalid, and fail only after both paths are exhausted.
12. Downloaded `cloudflared` artifacts and related state should use the same mod-specific Minecraft-instance directory structure as the modern branch where practical, including `modflared/bin/` and `modflared/forced_tunnels.json`.
13. Deterministic local-port behavior should match the modern branch if practical. If 1.12.2 requires a different algorithm, it must remain deterministic per host and the divergence must be documented.
14. The config surface should preserve file names and semantics as closely as practical, especially `forced_tunnels.json`.
15. Server-list ping parity means full ping routing is required; cosmetic parity such as hostname presentation, MOTD shaping, and icon behavior is best-effort.
16. Required v1 test cases include direct IP entry, hostname entry, and saved server-list entries.
17. Behavior matters more than matching internal package and class layout.
18. Dependency replacements are allowed only as a last resort and must be documented.
19. The first milestone is satisfied by a buildable, launchable client with one working tunneled connection path.
20. Publishing workflow is out of scope for the first milestone.
21. The legacy line should preserve the same mod id and use an explicit legacy version marker, for example `1.12.2-legacy.1`.
22. The spec should include concrete 1.12.2 classes and methods only for the highest-risk hooks:

    * direct-connect hook
    * server-list ping hook
    * tunnel process lifecycle
    * config load path for `forced_tunnels.json`
23. No additional features are pre-approved as droppable exclusions beyond the default documented-gap policy.

## 17. Agent-facing implementation constraints derived from locked decisions

1. Use Forge `1.12.2-14.23.5.2860` unless an explicit maintainer decision changes it.
2. Preserve the modern deterministic local-port algorithm if practical. If a compatibility-specific algorithm is required on 1.12.2, it must remain deterministic per host and the divergence must be documented in `DECISIONS.md` and `API_DIFF_NOTES.md`.
3. Implement the `cloudflared` recovery chain in this order:

   * prefer local/system binary when valid
   * repair by downloading when local/system binary is invalid or unusable
   * fail only after both paths are exhausted
4. Store downloaded `cloudflared` artifacts and related state in the same mod-specific Minecraft-instance directory structure as the modern branch where practical, including `modflared/bin/` and `modflared/forced_tunnels.json`.
5. Treat server-list ping parity as required for routing behavior, but cosmetic parity such as hostname presentation, MOTD shaping, and icon behavior is best-effort.
6. Treat direct IP entry, hostname entry, and saved server-list entries as required test cases.
7. Preserve `forced_tunnels.json` filename and semantics as closely as practical.
8. Preserve the same mod id and use an explicit legacy version marker such as `1.12.2-legacy.1`.
9. Pin down concrete 1.12.2 classes and methods only for the highest-risk hooks. Do not try to over-specify the entire port before source inspection.
10. No extra droppable features are pre-authorized beyond the documented-gap policy.

## 18. Initial recommendation

For the first approved scope, prefer this boundary:

* Forge 1.12.2 only
* Java 8 only
* client-side only
* Windows and Linux first
* preserve `forced_tunnels.json` file naming where practical
* preserve the required local-tunnel traffic model
* require server-list ping support as part of target parity
* allow implementation divergence where 1.12.2 internals differ
* require a strict decision log and test plan alongside code changes

## 19. Next action

This document is now ready to be finalized as Draft v1 and split into:

* `TECH_SPEC.md`
* `TASKS.md`
* `TEST_PLAN.md`
* `DECISIONS.md`
* `API_DIFF_NOTES.md`
