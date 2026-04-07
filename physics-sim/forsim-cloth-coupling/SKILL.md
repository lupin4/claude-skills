---
name: "forsim-cloth-coupling"
description: "Use when Rigid-deformable coupling, PBD cloth, constraint stabilization (Baumgarte), friction models for cloth-rigid interaction."
---

# forSim Cloth Coupling

Rigid-deformable coupling, PBD cloth, constraint stabilization (Baumgarte), friction models for cloth-rigid interaction.

---

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
- [Implementation Patterns](#implementation-patterns)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

Provides patterns and conventions for forSim Cloth Coupling within the forSim/Newton physics simulation ecosystem. All physics kernels are implemented in Fortran with Zig orchestration, exporting to NVIDIA Newton via UsdPhysics.

### Prerequisites

- forSim Fortran kernels built
- OpenUSD Python API (for USD authoring skills)
- NVIDIA Newton (for scene assembly/export skills)

---

## Core Concepts

### Position-Based Dynamics (PBD) for Cloth

```fortran
! PBD step: predict → project constraints → update
subroutine fm_pbd_step(positions, velocities, n, dt, iterations, status) &
    bind(C, name="fm_pbd_step")
  integer(c_int), intent(in), value :: n, iterations
  real(c_double), intent(in), value :: dt
  real(c_double), intent(inout) :: positions(3, n), velocities(3, n)
  integer(c_int), intent(out) :: status
  ! 1. Predict: p* = p + v*dt + gravity*dt^2
  ! 2. Project distance constraints (iterations times)
  ! 3. Update: v = (p* - p_old) / dt
end subroutine
```

### Baumgarte Stabilization

Constraint drift correction: C_stabilized = C + β/dt * C_error

β = 0.1-0.3 typical for cloth-rigid contact

---

## Workflows

### Standard Workflow

1. Design scene/simulation parameters
2. Implement Fortran kernels with bind(C) exports
3. Create Zig wrappers and test harness
4. Export to USD (Newton-clean)
5. Validate in Newton

---

## Anti-Patterns

- **PhysxSchema in Newton exports** — use UsdPhysics only
- **Hardcoded physics parameters** — make configurable via scene USD
- **Skipping broadphase** — always use SAP or grid for > 10 bodies
- **Non-SI units** — everything in meters, kg, seconds, radians

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [3d-vfx/](../../3d-vfx/) — Blender export pipelines
- [synthetic-data/](../../synthetic-data/) — Scene generation
