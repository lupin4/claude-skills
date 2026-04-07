---
name: "openusd-schema"
description: "Use when working with Prim composition arcs (references, payloads, inherits, variants, specializes), layer stacking, UsdPhysics authoring, Python USD API."
---

# OpenUSD Schema

Prim composition arcs (references, payloads, inherits, variants, specializes), layer stacking, UsdPhysics authoring, Python USD API. Part of the 3D/VFX pipeline for robotics simulation and synthetic data generation.

---

## Overview

## Composition Arcs (LIVRPS order)

| Arc | Purpose | Syntax |
|-----|---------|--------|
| **L**ocal | Direct opinions on prim | Default |
| **I**nherits | Share opinions from class prim | inherits = </Class> |
| **V**ariants | Switchable alternatives | variantSets = "style" |
| **R**eferences | Compose from another layer/prim | references = @file.usd@ |
| **P**ayloads | Lazy-loaded references | payload = @heavy.usd@ |
| **S**pecializes | Like inherits, weakest strength | specializes = </Base> |

## UsdPhysics for Robotics

```python
from pxr import Usd, UsdPhysics, UsdGeom, Gf

stage = Usd.Stage.CreateNew("robot_scene.usda")
UsdGeom.SetStageUpAxis(stage, UsdGeom.Tokens.y)
UsdPhysics.SetStageKilogramsPerUnit(stage, 1.0)

# Physics scene
physics_scene = UsdPhysics.Scene.Define(stage, "/PhysicsScene")
physics_scene.CreateGravityDirectionAttr(Gf.Vec3f(0, -1, 0))
physics_scene.CreateGravityMagnitudeAttr(9.81)

# Ground plane
ground = UsdGeom.Mesh.Define(stage, "/World/Ground")
UsdPhysics.CollisionAPI.Apply(ground.GetPrim())
# Static body (no RigidBodyAPI = kinematic/static)

# Dynamic box
box = UsdGeom.Cube.Define(stage, "/World/Box")
UsdPhysics.RigidBodyAPI.Apply(box.GetPrim())
UsdPhysics.CollisionAPI.Apply(box.GetPrim())
mass = UsdPhysics.MassAPI.Apply(box.GetPrim())
mass.CreateMassAttr(1.0)

# Revolute joint
joint = UsdPhysics.RevoluteJoint.Define(stage, "/World/Joint_1")
joint.CreateBody0Rel().SetTargets(["/World/Ground"])
joint.CreateBody1Rel().SetTargets(["/World/Box"])
joint.CreateAxisAttr("X")
joint.CreateLowerLimitAttr(-90.0)
joint.CreateUpperLimitAttr(90.0)

stage.GetRootLayer().Save()
```

---

## Anti-Patterns

- **PhysxSchema in exports** — Newton-clean means UsdPhysics only
- **GUI-dependent scripts** — headless scripts must not use bpy.ops that need a viewport
- **Missing ACES config** — always configure OCIO for color-accurate rendering

---

## Cross-References

- [physics-sim/](../../physics-sim/) — Newton scene assembly, USD physics
- [synthetic-data/](../../synthetic-data/) — Render pipeline, interaction generation
