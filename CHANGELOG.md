# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
