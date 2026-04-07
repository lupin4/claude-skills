# Physics / Simulation — Claude Code Guidance

Physics simulation skills covering rigid body dynamics (forSim), cloth coupling, differentiable physics, and NVIDIA Newton scene authoring via USD.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **forsim-rigid-body** | Broadphase SAP, restitution/damping, deactivation thresholds |
| **forsim-cloth-coupling** | Rigid-deformable coupling, PBD cloth, constraint formulation |
| **forsim-usd-physics** | UsdPhysics schema authoring — NO PhysxSchema (Newton-clean) |
| **forsim-differentiable** | Forward/reverse AD for physics, Jacobians, forTorch integration |
| **newton-scene-assembly** | NVIDIA Newton scene graph conventions, joint/body naming |
| **newton-cloth-rigid** | Combined cloth + rigid USD export pipeline for Newton |

## Key Conventions

- **Physics engine:** forSim (Fortran kernels + Zig orchestration)
- **Export target:** NVIDIA Newton via UsdPhysics (never PhysxSchema)
- **Units:** SI (meters, kilograms, seconds, radians)
- **Coordinate system:** Right-hand, Y-up for USD, Z-up converted from Blender
- **Solver:** Iterative PGS for constraints, explicit Euler or Verlet for integration

## Cross-References

- 3d-vfx/ — Blender export pipelines, USD schema authoring
- synthetic-data/ — Scene generation for training data
- fortran-kernels/fortran-lie-groups — SO(3)/SE(3) for rigid body math
- ai-ml-kernels/forsim-differentiable ↔ fortorch-tensor-model
