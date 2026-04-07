# 3D / VFX Pipeline — Claude Code Guidance

Blender, USD, and VFX pipeline skills for robotics simulation and synthetic data — headless scripting, rig conventions, USD export, MetaHuman integration, and procedural variation.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **blender-headless-bpy** | Headless bpy scripting, physics bake, batch export |
| **blender-rig-conventions** | Joint axis patterns, LIMIT_ROTATION, Franka/KUKA/humanoid rigs |
| **blender-usd-clean** | UsdPhysics-only export, ACES color, no PhysxSchema |
| **openusd-schema** | Composition arcs, layer stacking, UsdPhysics authoring |
| **metahuman-rig** | Control Rig, MetaHuman bones, .glb bake, facial retarget |
| **geometry-nodes-syndata** | Procedural variation via Geometry Nodes for synthetic data |

## Key Conventions

- **Blender version:** 4.0+ (USD export improvements)
- **Invocation:** Always `blender --background --python script.py` for automation
- **Coordinate system:** Blender Z-up → USD Y-up conversion on export
- **USD target:** NVIDIA Newton — UsdPhysics only, never PhysxSchema
- **Color:** ACES 1.2 via OCIO throughout pipeline
- **Rig naming:** DEF-* (deform), MCH-* (mechanism), ORG-* (original)

## Cross-References

- physics-sim/ — Newton scene assembly, USD physics authoring
- synthetic-data/ — Interaction generation, render pipeline, URDF export
- fortran-kernels/fortran-lie-groups — SO(3) math for joint/rig transforms
