---
name: "syndata-urdf-remap"
description: "Use when Blender to URDF axis remapping (URDF_x=Blender_y), joint limit conventions, inertia tensor export, collision mesh simplification."
---

# Synthetic Data URDF Remap

Blender to URDF axis remapping (URDF_x=Blender_y), joint limit conventions, inertia tensor export, collision mesh simplification.

---

## Overview

### What This Skill Does

Part of the robotics synthetic data pipeline. Generates training data using headless Blender for robot manipulation tasks targeting NVIDIA Newton simulation.

## Axis Remapping Matrix

| URDF Axis | Blender Axis | Transform |
|-----------|-------------|-----------|
| X (forward) | Y | URDF_x = Blender_y |
| Y (left) | -X | URDF_y = -Blender_x |
| Z (up) | Z | URDF_z = Blender_z |

## Joint Export Convention

```python
# Blender bone → URDF joint
def bone_to_urdf_joint(bone):
    joint = {
        "name": bone.name.replace("DEF-", ""),
        "type": classify_joint_type(bone),  # revolute, prismatic, fixed
        "parent_link": bone.parent.name if bone.parent else "base_link",
        "child_link": bone.name,
        "origin": {
            "xyz": remap_position(bone.head),  # Apply axis remap
            "rpy": remap_rotation(bone.matrix_local),
        },
        "axis": remap_axis(get_rotation_axis(bone)),
        "limit": get_joint_limits(bone),
    }
    return joint

def remap_position(blender_pos):
    """Blender (x,y,z) → URDF (y, -x, z)"""
    return [blender_pos.y, -blender_pos.x, blender_pos.z]
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
