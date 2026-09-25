# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.4.1] - 2026-09-25

### Fixed
- The README adds the library with `mach dep add` and shows the stanza it writes: `[dep.phys]` at `version = "^0.4.0"`, where it showed the invalid `[deps.mach-phys]` key at `ref = "branch/main"`. It also describes the CI the repo runs today (#32).

## [0.4.0] - 2026-09-25

### Changed
- **Breaking: builds against std 8.0.0 and requires mach 5.12** (#33). `[dep.std]` moves from the exact `ref = "tag/v4.0.0"` to the range `version = "^8.0"`, realized to v8.0.0 by the committed `dep/std` gitlink, and `[project].mach` rises from `^5.3` to `^5.12`, which std 8 requires. Resolution is flat, so a consumer of phys must move to std 8 and mach 5.12 with it, and must rebuild anything that links std rather than only recompiling against the new sources. A range instead of an exact tag means a root on a later std 8 minor no longer conflicts with phys. No source change was needed across std 5, 6, 7 and 8: phys imports only `std.runtime` and `std.types`, which none of them reshaped. Every module that holds a test is reached from `phys.mach`, so mach 5.12's closure-scoped `mach test .` (briar-systems/mach#3813) still collects all 13 tests on every target.
- ci: the lib job seeds mach v5.12.0 until the family pin moves (briar-systems/.github#103) (#33).
- manifest: declares the compiler range `mach = "^5.3"`, so mach 5.3 and later no longer warn on every build.
- license: copyright is attributed to Briar Systems LLC.
- ci: releases publish through the family release workflow (`briar-systems/.github` `mach-release.yml`). Pushing a `v*` tag verifies the tag against the manifest and changelog, runs the full CI tier and publishes the GitHub release.

## [0.3.1] - 2026-09-16

### Changed
- deps: std moves to `tag/v4.0.0`, which requires mach 5.2.0 or later. phys uses none of the names 4.0.0 removed, so no source changes were needed.

## [0.3.0] - 2026-09-16

### Added
- manifest: `linux-arm64` and `darwin-aarch64` targets, so the native aarch64 hosts build and test for themselves instead of falling back to linux-x86_64.

### Changed
- builds with Mach 5.0 and std 3.2 (`tag/v3.2.0`). The dependency is `[dep.std]`, pinned by the committed `dep/std` gitlink, and `mach.lock` is gone. No source changes were needed for the std 3.x io runtime.
- ci: CI runs the family pipeline (`briar-systems/.github` `mach-lib.yml`) on the pinned, checksum-verified mach seed: debug and release build and test, `mach fmt --check` and an all-targets release build on x86_64-linux for pull requests into dev, plus native aarch64-linux, windows and darwin legs for pull requests into main. A `gate` job is the one required check.
- style: source reformatted with `mach fmt`.

## [0.2.0] - 2026-08-07

First working simulation. The scaffold deferred implementation behind Mach's
built-in SIMD vector types so that no interim math shim would be written. Those
types have landed, so the vector math is `f32x4` directly.

### Added
- `phys.vec` — `Vec3` as an `f32x4` with lane 3 unused.
- `phys.body` — point masses with AABB half-extents. Mass is stored inverted, so a static body has `inv_mass` 0. That drops out of the impulse arithmetic without a branch and keeps division out of the step.
- `phys.world` — semi-implicit integration, an all-pairs broad phase, and impulse resolution with positional correction. Contacts found during a step are reported for the caller to consume.

### Scope
Rotation is not modelled: AABBs do not rotate, so orientation would be state that nothing reads. Joints and continuous collision are also out. `collect_contacts` is the only function that enumerates pairs, so a grid or BVH replaces it alone.

### Changed
- manifest: Re-touched to RFC-exact totality per mach#1964/mach#1979, and gained the `simd` profile key Mach now requires.

### Verification
13 tests, including an integration case that drops a body for 200 steps and asserts it rests on static ground. Three decisions were checked against deliberately broken variants: explicit Euler fails the gravity test, selecting the deepest separating axis instead of the shallowest fails three tests, and removing positional correction lets a falling body sink through the ground.

## [0.1.0] - 2026-07-07

### Added
- Initial release of mach-phys.

### Changed
- manifest: Migrated manifest to v2 schema (`[artifact.phys]`).
- deps: Updated `mach-std` dependency to the git URL.
