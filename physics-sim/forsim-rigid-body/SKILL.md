---
name: "forsim-rigid-body"
description: "Use when Broadphase SAP, restitution/damping tuning, deactivation thresholds, island detection for rigid body simulation."
---

# forSim Rigid Body

Broadphase SAP, restitution/damping tuning, deactivation thresholds, island detection for rigid body simulation.

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

Provides patterns and conventions for forSim Rigid Body within the forSim/Newton physics simulation ecosystem. All physics kernels are implemented in Fortran with Zig orchestration, exporting to NVIDIA Newton via UsdPhysics.

### Prerequisites

- forSim Fortran kernels built
- OpenUSD Python API (for USD authoring skills)
- NVIDIA Newton (for scene assembly/export skills)

---

## Core Concepts

### Broadphase: Sweep and Prune (SAP)

SAP maintains sorted lists of AABB projections on each axis. Overlaps detected by sweep-line.

```fortran
! AABB structure
type, bind(C) :: ft_aabb_t
  real(c_double) :: min_x, min_y, min_z
  real(c_double) :: max_x, max_y, max_z
end type

! SAP sorts endpoints, finds overlapping pairs
subroutine fm_sap_broadphase(aabbs, n, pairs, npairs, status) bind(C, name="fm_sap_broadphase")
  integer(c_int), intent(in), value :: n
  type(ft_aabb_t), intent(in) :: aabbs(n)
  integer(c_int), intent(out) :: pairs(2, n*n/2)  ! worst case
  integer(c_int), intent(out) :: npairs, status
  ! Sort on X axis, sweep for overlaps, prune with Y and Z
end subroutine
```

### Tuning Parameters

| Parameter | Typical Range | Effect |
|-----------|--------------|--------|
| Restitution (e) | 0.0 - 1.0 | 0 = perfectly inelastic, 1 = perfectly elastic |
| Linear damping | 0.0 - 0.5 | Velocity decay per second |
| Angular damping | 0.0 - 0.5 | Angular velocity decay |
| Sleep threshold (linear) | 0.01 - 0.1 m/s | Below this → candidate for deactivation |
| Sleep threshold (angular) | 0.01 - 0.1 rad/s | Below this → candidate for deactivation |
| Sleep time | 0.5 - 2.0 s | Duration below threshold before sleep |

### Island Detection

Connected components of interacting bodies. Sleeping islands skip simulation entirely.

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
