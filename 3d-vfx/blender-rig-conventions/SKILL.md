---
name: "blender-rig-conventions"
description: "Use when working with Bone naming (DEF-*, MCH-*, ORG-*), joint axis alignment for robotics, LIMIT_ROTATION constraints, Franka/KUKA/humanoid rigs."
---

# Blender Rig Conventions

Bone naming (DEF-*, MCH-*, ORG-*), joint axis alignment for robotics, LIMIT_ROTATION constraints, Franka/KUKA/humanoid rigs. Part of the 3D/VFX pipeline for robotics simulation and synthetic data generation.

---

## Overview

## Bone Naming Convention

| Prefix | Purpose | Example |
|--------|---------|---------|
| DEF- | Deform bones (weighted to mesh) | DEF-upper_arm.L |
| MCH- | Mechanism bones (constraints/drivers) | MCH-elbow_target.L |
| ORG- | Original reference bones | ORG-upper_arm.L |
| (none) | Control bones (animator-facing) | upper_arm_fk.L |

## Robot Rig Axis Mapping

| Robot Joint Axis | Blender Bone Axis | Constraint |
|-----------------|-------------------|-----------|
| Rotation around Z (typical) | Local Y (bone axis) | LIMIT_ROTATION on Y |
| Rotation around X | Local X | LIMIT_ROTATION on X |
| Prismatic along Z | Local Y translation | LIMIT_LOCATION on Y |

## LIMIT_ROTATION Setup

```python
import bpy
import math

def setup_robot_joint(armature_name, bone_name, lower_deg, upper_deg, axis='Y'):
    arm = bpy.data.objects[armature_name]
    bone = arm.pose.bones[bone_name]
    
    # Clear existing constraints
    for c in bone.constraints:
        if c.type == 'LIMIT_ROTATION':
            bone.constraints.remove(c)
    
    constraint = bone.constraints.new('LIMIT_ROTATION')
    constraint.owner_space = 'LOCAL'
    
    lower = math.radians(lower_deg)
    upper = math.radians(upper_deg)
    
    setattr(constraint, f'use_limit_{axis.lower()}', True)
    setattr(constraint, f'min_{axis.lower()}', lower)
    setattr(constraint, f'max_{axis.lower()}', upper)
```

## Franka Panda Rig (7-DOF)

```
robot_franka
├── base_link (fixed)
├── link_1 → joint_1 (revolute, Z axis, ±166°)
├── link_2 → joint_2 (revolute, Z axis, ±101°)
├── link_3 → joint_3 (revolute, Z axis, ±166°)
├── link_4 → joint_4 (revolute, Z axis, -176° to -4°)
├── link_5 → joint_5 (revolute, Z axis, ±166°)
├── link_6 → joint_6 (revolute, Z axis, -1° to 215°)
├── link_7 → joint_7 (revolute, Z axis, ±166°)
└── hand → finger joints (prismatic, 0-0.04m)
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
