---
name: "syndata-render-pipeline"
description: "Use when ACES 1.2/OCIO setup in Blender, Cycles vs EEVEE for syndata, render resolution/samples, .glb bake, multi-pass (depth, normals, segmentation masks)."
---

# Synthetic Data Render Pipeline

ACES 1.2/OCIO setup in Blender, Cycles vs EEVEE for syndata, render resolution/samples, .glb bake, multi-pass (depth, normals, segmentation masks).

---

## Overview

### What This Skill Does

Part of the robotics synthetic data pipeline. Generates training data using headless Blender for robot manipulation tasks targeting NVIDIA Newton simulation.

## ACES 1.2 Configuration

```python
import bpy
bpy.context.scene.display_settings.display_device = 'ACES'
bpy.context.scene.view_settings.view_transform = 'ACES 1.0 SDR-video'
bpy.context.scene.sequencer_colorspace_settings.name = 'ACES - ACEScg'
```

## Multi-Pass Render Setup

```python
scene = bpy.context.scene
scene.use_nodes = True
tree = scene.node_tree

# Enable passes
view_layer = bpy.context.view_layer
view_layer.use_pass_z = True          # Depth
view_layer.use_pass_normal = True     # Normals
view_layer.use_pass_object_index = True  # Segmentation

# Render settings for synthetic data
scene.render.engine = 'CYCLES'
scene.cycles.samples = 128           # Enough for clean syndata
scene.render.resolution_x = 1280
scene.render.resolution_y = 720
scene.render.image_settings.file_format = 'PNG'
scene.render.image_settings.color_depth = '16'
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
