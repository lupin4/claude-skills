---
name: "blender-usd-clean"
description: "Use when working with UsdPhysics-only export (no PhysxSchema), ACES 1.2 color pipeline, UsdPreviewSurface materials, Newton-clean export."
---

# Blender USD Clean Export

UsdPhysics-only export (no PhysxSchema), ACES 1.2 color pipeline, UsdPreviewSurface materials, Newton-clean export. Part of the 3D/VFX pipeline for robotics simulation and synthetic data generation.

---

## Overview

## Newton-Clean USD Export

```python
import bpy

# Configure export settings
bpy.ops.wm.usd_export(
    filepath="/path/to/scene.usda",
    export_animation=True,
    export_hair=False,
    export_uvmaps=True,
    export_normals=True,
    export_materials=True,
    use_instancing=True,
    evaluation_mode='RENDER',
)
```

## Post-Export Cleanup (Remove PhysxSchema)

```python
from pxr import Usd, UsdPhysics

stage = Usd.Stage.Open("scene.usda")
for prim in stage.Traverse():
    # Remove any PhysxSchema attributes
    for attr in prim.GetAttributes():
        if "physxSchema" in attr.GetName() or "PhysxSchema" in attr.GetName():
            prim.RemoveProperty(attr.GetName())

# Apply UsdPhysics APIs properly
for prim in stage.Traverse():
    if prim.HasAPI(UsdPhysics.RigidBodyAPI):
        # Ensure mass is set
        mass_api = UsdPhysics.MassAPI.Apply(prim)
        if not mass_api.GetMassAttr().HasValue():
            mass_api.CreateMassAttr(1.0)

stage.GetRootLayer().Save()
```

## ACES 1.2 Setup in Blender

```python
# Set OCIO config for ACES
import os
os.environ['OCIO'] = '/path/to/aces/config.ocio'

scene = bpy.context.scene
scene.display_settings.display_device = 'ACES'
scene.view_settings.view_transform = 'ACES 1.0 SDR-video'
scene.sequencer_colorspace_settings.name = 'ACES - ACEScg'
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
