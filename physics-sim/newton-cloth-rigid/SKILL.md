---
name: "newton-cloth-rigid"
description: "Use when Combined cloth mesh + rigid body USD export pipeline for NVIDIA Newton ingestion, export validation, USD layer organization."
---

# Newton Cloth-Rigid Export

Combined cloth mesh + rigid body USD export pipeline for NVIDIA Newton ingestion, export validation, USD layer organization.

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

Provides patterns and conventions for Newton Cloth-Rigid Export within the forSim/Newton physics simulation ecosystem. All physics kernels are implemented in Fortran with Zig orchestration, exporting to NVIDIA Newton via UsdPhysics.

### Prerequisites

- forSim Fortran kernels built
- OpenUSD Python API (for USD authoring skills)
- NVIDIA Newton (for scene assembly/export skills)

---

## Core Concepts

### Combined Export Pipeline

```python
# 1. Author rigid bodies
rigid = UsdGeom.Mesh.Define(stage, "/World/Table")
UsdPhysics.RigidBodyAPI.Apply(rigid.GetPrim())
UsdPhysics.CollisionAPI.Apply(rigid.GetPrim())

# 2. Author cloth mesh
cloth_mesh = UsdGeom.Mesh.Define(stage, "/World/Cloth")
# DeformableBodyAPI for soft body / cloth
# Note: Newton uses its own cloth solver — export mesh + constraints

# 3. Material binding
material = UsdShade.Material.Define(stage, "/World/Materials/Fabric")
UsdShade.MaterialBindingAPI(cloth_mesh).Bind(material)

# 4. Validate
assert not any("PhysxSchema" in str(p) for p in stage.Traverse())
```

### Export Validation Checklist

- [ ] No PhysxSchema attributes anywhere in stage
- [ ] All rigid bodies have RigidBodyAPI + CollisionAPI
- [ ] All joints have body0/body1 relationships set
- [ ] PhysicsScene has gravity configured
- [ ] Units are SI (meters, kg, seconds)

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
