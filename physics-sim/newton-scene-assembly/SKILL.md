---
name: "newton-scene-assembly"
description: "Use when NVIDIA Newton scene graph conventions, joint/body naming, material assignment, collision groups, solver settings for robot simulation."
---

# Newton Scene Assembly

NVIDIA Newton scene graph conventions, joint/body naming, material assignment, collision groups, solver settings for robot simulation.

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

Provides patterns and conventions for Newton Scene Assembly within the forSim/Newton physics simulation ecosystem. All physics kernels are implemented in Fortran with Zig orchestration, exporting to NVIDIA Newton via UsdPhysics.

### Prerequisites

- forSim Fortran kernels built
- OpenUSD Python API (for USD authoring skills)
- NVIDIA Newton (for scene assembly/export skills)

---

## Core Concepts

### Scene Hierarchy Convention

```
/World
  /World/PhysicsScene          # UsdPhysics.Scene (gravity, solver)
  /World/GroundPlane           # Static rigid body
  /World/Robot
    /World/Robot/base_link     # Root link
    /World/Robot/link_1        # First joint child
    /World/Robot/joint_1       # RevoluteJoint connecting base↔link_1
    ...
  /World/Objects
    /World/Objects/box_01      # Dynamic rigid body
```

### Naming Convention

| Element | Pattern | Example |
|---------|---------|---------|
| Robot links | `{robot_name}/link_{N}` | `franka/link_3` |
| Joints | `{robot_name}/joint_{N}` | `franka/joint_3` |
| Objects | `objects/{type}_{NN}` | `objects/box_01` |
| Ground | `ground_plane` | `/World/ground_plane` |
| Scene | `PhysicsScene` | `/World/PhysicsScene` |

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
