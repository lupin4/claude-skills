---
name: "blender-headless-bpy"
description: "Use when working with blender --background --python scripting, bpy.context/bpy.data in headless mode, scene setup, physics bake, batch export (USD, glTF, FBX)."
---

# Blender Headless bpy

blender --background --python scripting, bpy.context/bpy.data in headless mode, scene setup, physics bake, batch export (USD, glTF, FBX). Part of the 3D/VFX pipeline for robotics simulation and synthetic data generation.

---

## Overview

## Headless Invocation

```bash
blender --background --python script.py -- --arg1 value1 --arg2 value2
```

### Script Template

```python
import bpy
import sys

def main():
    # Parse custom args (after --)
    argv = sys.argv
    args = argv[argv.index("--") + 1:] if "--" in argv else []
    
    # Clear default scene
    bpy.ops.wm.read_factory_settings(use_empty=True)
    
    # Setup scene
    scene = bpy.context.scene
    scene.render.engine = 'CYCLES'
    scene.cycles.device = 'GPU'
    scene.cycles.samples = 128
    
    # Add camera
    cam_data = bpy.data.cameras.new("Camera")
    cam_obj = bpy.data.objects.new("Camera", cam_data)
    scene.collection.objects.link(cam_obj)
    scene.camera = cam_obj
    
    # Add light
    light_data = bpy.data.lights.new("Light", 'SUN')
    light_obj = bpy.data.objects.new("Light", light_data)
    scene.collection.objects.link(light_obj)
    
    # Import/create objects...
    
    # Render
    scene.render.filepath = "/tmp/render_output"
    bpy.ops.render.render(write_still=True)

if __name__ == "__main__":
    main()
```

## Common Headless Pitfalls

| Issue | Cause | Fix |
|-------|-------|-----|
| No GPU rendering | No display | Use CYCLES with GPU compute |
| bpy.context empty | No active object in headless | Set explicitly: bpy.context.view_layer.objects.active = obj |
| Import fails silently | Missing file path | Use absolute paths, check os.path.exists |
| Memory exhaustion | Large scenes | Purge orphan data: bpy.ops.outliner.orphans_purge() |

---

## Anti-Patterns

- **PhysxSchema in exports** — Newton-clean means UsdPhysics only
- **GUI-dependent scripts** — headless scripts must not use bpy.ops that need a viewport
- **Missing ACES config** — always configure OCIO for color-accurate rendering

---

## Cross-References

- [physics-sim/](../../physics-sim/) — Newton scene assembly, USD physics
- [synthetic-data/](../../synthetic-data/) — Render pipeline, interaction generation
