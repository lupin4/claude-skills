# Robotics Synthetic Data — Claude Code Guidance

Synthetic data generation skills for robotics training — headless Blender scene generation, parameter variation, dataset curation, and robot rig conventions.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **syndata-interaction-gen** | interaction_gen.py conventions, rig naming, interaction types |
| **syndata-variation-engine** | Parameter sweeps, seed management, variation taxonomy |
| **syndata-dataset-curator** | Output schema, annotation formats, dataset versioning |
| **syndata-render-pipeline** | ACES/OCIO, Cycles rendering, multi-pass output, .glb bake |
| **syndata-urdf-remap** | Blender→URDF axis remapping, joint conventions |
| **syndata-kuka-franka** | Franka Panda + Kuka iiwa joint axis patterns, LIMIT_ROTATION |

## Key Conventions

- **Renderer:** Blender Cycles headless (`blender --background --python`)
- **Color pipeline:** ACES 1.2 via OCIO
- **Rig naming:** `robot_*`, `object_*`, `surface_*` prefixes
- **Axis convention:** Blender Z-up → URDF (URDF_x = Blender_y, URDF_y = -Blender_x, URDF_z = Blender_z)
- **Reproducibility:** All randomization seeded, seeds logged in metadata
- **Annotation:** COCO JSON primary, PASCAL VOC secondary

## Cross-References

- 3d-vfx/ — Blender headless scripting, rig conventions, USD export
- physics-sim/ — Newton scene assembly for simulated interactions
- fortran-kernels/fortran-lie-groups — SO(3) for pose randomization
