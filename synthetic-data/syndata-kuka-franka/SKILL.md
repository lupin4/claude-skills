---
name: "syndata-kuka-franka"
description: "Use when Franka Panda 7-DOF + Kuka iiwa joint conventions, LIMIT_ROTATION constraints in Blender, joint axis alignment, torque limits."
---

# Synthetic Data Kuka/Franka

Franka Panda 7-DOF + Kuka iiwa joint conventions, LIMIT_ROTATION constraints in Blender, joint axis alignment, torque limits.

---

## Overview

### What This Skill Does

Part of the robotics synthetic data pipeline. Generates training data using headless Blender for robot manipulation tasks targeting NVIDIA Newton simulation.

## Franka Emika Panda — 7-DOF Joint Specs

| Joint | Type | Range (rad) | Max Torque (Nm) | Max Velocity (rad/s) |
|-------|------|------------|----------------|---------------------|
| 1 | Revolute | [-2.8973, 2.8973] | 87 | 2.175 |
| 2 | Revolute | [-1.7628, 1.7628] | 87 | 2.175 |
| 3 | Revolute | [-2.8973, 2.8973] | 87 | 2.175 |
| 4 | Revolute | [-3.0718, -0.0698] | 87 | 2.175 |
| 5 | Revolute | [-2.8973, 2.8973] | 12 | 2.610 |
| 6 | Revolute | [-0.0175, 3.7525] | 12 | 2.610 |
| 7 | Revolute | [-2.8973, 2.8973] | 12 | 2.610 |

## Kuka iiwa 14 — 7-DOF Joint Specs

| Joint | Type | Range (rad) | Max Torque (Nm) |
|-------|------|------------|----------------|
| 1 | Revolute | [-2.967, 2.967] | 320 |
| 2 | Revolute | [-2.094, 2.094] | 320 |
| 3 | Revolute | [-2.967, 2.967] | 176 |
| 4 | Revolute | [-2.094, 2.094] | 176 |
| 5 | Revolute | [-2.967, 2.967] | 110 |
| 6 | Revolute | [-2.094, 2.094] | 40 |
| 7 | Revolute | [-3.054, 3.054] | 40 |

## Blender LIMIT_ROTATION Setup

```python
import bpy

def apply_joint_limits(armature_name, bone_name, axis, lower, upper):
    arm = bpy.data.objects[armature_name]
    bone = arm.pose.bones[bone_name]
    
    constraint = bone.constraints.new('LIMIT_ROTATION')
    constraint.owner_space = 'LOCAL'
    
    if axis == 'X':
        constraint.use_limit_x = True
        constraint.min_x = lower
        constraint.max_x = upper
    elif axis == 'Y':
        constraint.use_limit_y = True
        constraint.min_y = lower
        constraint.max_y = upper
    elif axis == 'Z':
        constraint.use_limit_z = True
        constraint.min_z = lower
        constraint.max_z = upper
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
