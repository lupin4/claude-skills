---
name: "metahuman-rig"
description: "Use when working with MetaHuman face rig structure, Control Rig node graph, bone hierarchy, LOD management, .glb export with baked animations."
---

# MetaHuman Rig

MetaHuman face rig structure, Control Rig node graph, bone hierarchy, LOD management, .glb export with baked animations. Part of the 3D/VFX pipeline for robotics simulation and synthetic data generation.

---

## Overview

## MetaHuman Face Rig Structure

```
root
├── spine_05 (head)
│   ├── FACIAL_C_FacialRoot
│   │   ├── FACIAL_C_Jaw
│   │   ├── FACIAL_L_Eye
│   │   ├── FACIAL_R_Eye
│   │   ├── FACIAL_C_Nose
│   │   ├── FACIAL_C_Mouth
│   │   └── ... (200+ facial bones)
│   ├── FACIAL_C_Teeth_Upper
│   └── FACIAL_C_Teeth_Lower
```

## Control Rig Node Patterns

| Node Type | Purpose |
|-----------|---------|
| Hierarchy | Parent/child bone chains |
| Math | Float/vector math for blending |
| Transform | Set/get bone transforms |
| Animation | Pose caching, blending |

## .glb Export with Baked Animations

```python
# Select MetaHuman mesh and armature
bpy.ops.object.select_all(action='DESELECT')
armature = bpy.data.objects['Armature']
mesh = bpy.data.objects['Head']
armature.select_set(True)
mesh.select_set(True)
bpy.context.view_layer.objects.active = armature

# Bake animation
bpy.ops.nla.bake(
    frame_start=1,
    frame_end=250,
    only_selected=True,
    visual_keying=True,
    clear_constraints=True,
    bake_types={'POSE'}
)

# Export
bpy.ops.export_scene.gltf(
    filepath="/path/to/metahuman.glb",
    export_format='GLB',
    export_animations=True,
    export_skins=True,
)
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
