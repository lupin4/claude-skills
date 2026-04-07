---
name: "forsim-usd-physics"
description: "Use when UsdPhysics schema authoring with NO PhysxSchema dependency. Clean export for NVIDIA Newton — RigidBodyAPI, CollisionAPI, MassAPI, joint schemas."
---

# forSim USD Physics

UsdPhysics schema authoring with NO PhysxSchema dependency. Clean export for NVIDIA Newton — RigidBodyAPI, CollisionAPI, MassAPI, joint schemas.

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

Provides patterns and conventions for forSim USD Physics within the forSim/Newton physics simulation ecosystem. All physics kernels are implemented in Fortran with Zig orchestration, exporting to NVIDIA Newton via UsdPhysics.

### Prerequisites

- forSim Fortran kernels built
- OpenUSD Python API (for USD authoring skills)
- NVIDIA Newton (for scene assembly/export skills)

---

## Core Concepts

### UsdPhysics API Application

```python
from pxr import Usd, UsdPhysics, UsdGeom, Gf

stage = Usd.Stage.CreateNew("scene.usda")

# Rigid body
cube = UsdGeom.Cube.Define(stage, "/World/Cube")
UsdPhysics.RigidBodyAPI.Apply(cube.GetPrim())
UsdPhysics.CollisionAPI.Apply(cube.GetPrim())
mass_api = UsdPhysics.MassAPI.Apply(cube.GetPrim())
mass_api.CreateMassAttr(1.0)

# Joint
joint = UsdPhysics.RevoluteJoint.Define(stage, "/World/Joint")
joint.CreateBody0Rel().SetTargets(["/World/Base"])
joint.CreateBody1Rel().SetTargets(["/World/Arm"])
joint.CreateAxisAttr("X")
joint.CreateLowerLimitAttr(-90.0)
joint.CreateUpperLimitAttr(90.0)
```

### Newton-Clean Export Rules

1. **NEVER** use PhysxSchema — Newton has its own solver
2. Use ONLY UsdPhysics API classes
3. Set scene-level gravity via UsdPhysics.Scene
4. Use UsdPhysics.CollisionGroup for filtering

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
