# mach-phys

Pure-mach rigid-body physics for both 2D **and** 3D, in one library. The two
dimensions are not separate engines bolted together: they share a
dimension-generic core, and only the genuinely dimension-specific pieces are
written twice. Project id is `phys`, so consumers reach everything as `phys.*`.

Consuming projects vendor it as a normal Mach dependency. It links nothing and
forces no `libs` on consumers — the whole library is pure algorithms.

```toml
[deps.mach-phys]
git = "https://github.com/briar-systems/mach-phys"
ref = "branch/main"
```

> **Status: scaffold.** Implementation is intentionally sequenced behind
> Mach's upcoming built-in SIMD vector types. The vector and matrix math this
> library is built on will be first-class language types, so no interim math
> shim will be written — the engine is authored directly against the real
> types once they land. This repository currently fixes the project's
> identity, layout, and scope; the solver is not yet implemented.

## Design

One library, two dimensions, a shared spine:

- **Shared, dimension-generic core** — broadphase, island management, and the
  impulse-solver loop are written once and parameterized over dimension. The
  pipeline (pair-finding, contact islands, sequential-impulse resolution,
  integration) is identical in shape whether the vectors are 2- or 3-wide.
- **Per-dimension pieces** — narrowphase manifold generation and rotational
  dynamics are the parts that genuinely differ. 2D rotation is a scalar and a
  single moment of inertia; 3D rotation is a quaternion and an inertia tensor.
  Contact manifold generation is shape-pair specific and dimension specific.
  These are written per dimension; everything upstream and downstream is not.

3D scope is deliberately simple and honest about it:

- Shapes: spheres, boxes, capsules.
- Solver: impulse-based sequential (Gauss–Seidel) constraint resolution.
- Rotation: quaternion integration with an inertia tensor.

## Non-goals

These are held as firm boundaries, not "later" items. Crossing them turns a
small, comprehensible engine into a different project:

- **Mesh colliders** (arbitrary triangle-soup / convex-hull collision).
- **Continuous collision detection** (swept tests / tunneling prevention).
- **Large-stack stability guarantees** (deep, jitter-free box stacks).
- **Joint-rich constraint systems** (motors, ragdolls, extensive joint
  libraries).

Production-grade 3D physics with those properties is a **binding-beside-this-library**
problem — wrap a mature engine such as [Jolt](https://github.com/jrouwe/JoltPhysics)
in its own `mach-*` project — not scope creep inside `mach-phys`. This library
covers the common case (simple shapes, modest scenes, readable code) in pure
Mach; the heavyweight case belongs next to it, not inside it.

## Platforms

The library is pure algorithms with no OS dependencies, so it runs on every
Mach prime target:

| ISA | OS |
|---|---|
| `x86_64` | linux, darwin, windows |
| `aarch64` | linux, darwin |

There is no platform-specific code to maintain; portability follows from the
compiler's target support rather than from anything in this repository.

## Layout

```
src/
  phys.mach    library surface (phys.phys): the public entry point
```

The module tree will grow along the design above — a shared core plus
per-dimension narrowphase and rotational-dynamics modules — as the
SIMD-backed implementation lands. The surface stays flat: a bare `use phys;`
resolves to `phys.mach` and reaches the whole API as `phys.*`.

## Tests

`test` blocks live beside the code they cover and are display-free, run by
`mach test .`. CI fetches the latest released Mach compiler, syncs
dependencies, and runs both `mach build .` and `mach test .` on every pull
request into `dev` and `main`.
