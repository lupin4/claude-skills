---
name: "forsim-differentiable"
description: "Use when Forward/reverse AD for physics, Jacobians for rigid body dynamics, differentiable contact, gradient flow through constraint solvers."
---

# forSim Differentiable

Forward/reverse AD for physics, Jacobians for rigid body dynamics, differentiable contact, gradient flow through constraint solvers.

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

Provides patterns and conventions for forSim Differentiable within the forSim/Newton physics simulation ecosystem. All physics kernels are implemented in Fortran with Zig orchestration, exporting to NVIDIA Newton via UsdPhysics.

### Prerequisites

- forSim Fortran kernels built
- OpenUSD Python API (for USD authoring skills)
- NVIDIA Newton (for scene assembly/export skills)

---

## Core Concepts

### Forward-Mode AD for Physics

For rigid body simulation f(q, v, u) → (q', v'):

```
∂q'/∂u = ∂q'/∂v · ∂v'/∂u + ∂q'/∂u_direct
```

Jacobian of one timestep propagated forward through the simulation chain.

### Differentiable Contact

Contact forces are non-smooth. Approaches:
- **Soft contact:** Smooth penalty force, differentiable everywhere
- **Randomized smoothing:** Average over perturbations
- **Analytical subgradients:** At contact transitions

### forTorch Integration

Tensor memory model (view-based, refcounted) connects simulation gradients to ML training via shared Fortran kernel interface.

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
