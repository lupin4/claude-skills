---
name: "syndata-interaction-gen"
description: "Use when interaction_gen.py conventions: rig naming (robot_*, object_*, surface_*), interaction types (grasp, push, place, stack), headless Blender scripting."
---

# Synthetic Data Interaction Generator

interaction_gen.py conventions: rig naming (robot_*, object_*, surface_*), interaction types (grasp, push, place, stack), headless Blender scripting.

---

## Overview

### What This Skill Does

Part of the robotics synthetic data pipeline. Generates training data using headless Blender for robot manipulation tasks targeting NVIDIA Newton simulation.

## Rig Naming Convention

| Prefix | Type | Examples |
|--------|------|---------|
| `robot_` | Robot rigs | `robot_franka`, `robot_kuka_iiwa` |
| `object_` | Manipulable objects | `object_mug_01`, `object_box_03` |
| `surface_` | Work surfaces | `surface_table`, `surface_conveyor` |
| `camera_` | Camera rigs | `camera_overhead`, `camera_wrist` |

## Interaction Types

| Type | Description | Parameters |
|------|-------------|-----------|
| `grasp` | Gripper closes on object | approach_dir, grasp_width, force |
| `push` | Contact push without grasping | push_dir, push_dist, contact_point |
| `place` | Release object on surface | place_pos, place_rot, release_height |
| `stack` | Place on top of another object | target_object, alignment_tol |
| `pour` | Tilt container to pour | tilt_angle, pour_rate, target_vessel |

## Headless Blender Invocation

```bash
blender --background --python interaction_gen.py --   --robot robot_franka   --interaction grasp   --object object_mug_01   --surface surface_table   --seed 42   --output-dir /data/syndata/grasp_001
```

```python
# interaction_gen.py structure
import bpy
import sys

def main():
    args = parse_args(sys.argv[sys.argv.index("--") + 1:])
    setup_scene(args.surface)
    load_robot(args.robot)
    load_object(args.object)
    configure_interaction(args.interaction)
    randomize_params(args.seed)
    render_passes()
    export_annotations()

if __name__ == "__main__":
    main()
```

---

## Anti-Patterns

- **Non-reproducible randomization** — always seed everything, log seeds
- **Missing metadata** — every render must log all parameters
- **Hardcoded paths** — use CLI arguments for all paths
- **PhysxSchema in USD exports** — Newton-clean only

---

## Cross-References

- [3d-vfx/](../../3d-vfx/) — Blender headless, USD export
- [physics-sim/](../../physics-sim/) — Newton scene assembly
