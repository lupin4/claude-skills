---
name: "geometry-nodes-syndata"
description: "Use when working with Geometry Nodes for object scattering, material variation, procedural deformation, camera randomization, batch render parameter sweeps."
---

# Geometry Nodes Syndata

Geometry Nodes for object scattering, material variation, procedural deformation, camera randomization, batch render parameter sweeps. Part of the 3D/VFX pipeline for robotics simulation and synthetic data generation.

---

## Overview

## Object Scattering with Geometry Nodes

```python
# Create Geometry Nodes modifier for scattering
import bpy

obj = bpy.data.objects['Surface']
modifier = obj.modifiers.new('Scatter', 'NODES')

# Build node tree programmatically
tree = modifier.node_group
nodes = tree.nodes
links = tree.links

# Distribute Points on Faces
distribute = nodes.new('GeometryNodeDistributePointsOnFaces')
distribute.distribute_method = 'POISSON'
distribute.inputs['Density'].default_value = 10.0
distribute.inputs['Seed'].default_value = 42

# Instance on Points
instance = nodes.new('GeometryNodeInstanceOnPoints')

# Random rotation per instance
random_rot = nodes.new('FunctionNodeRandomValue')
random_rot.data_type = 'FLOAT_VECTOR'
```

## Parameter Sweeps for Synthetic Data

```python
# Batch render with varying parameters
import json

params = {
    'density': [5, 10, 20, 50],
    'seed': list(range(100)),
    'light_intensity': [500, 1000, 2000, 5000],
}

for density in params['density']:
    for seed in params['seed']:
        for intensity in params['light_intensity']:
            # Set Geometry Nodes parameters
            modifier.node_group.nodes['Distribute'].inputs['Density'].default_value = density
            modifier.node_group.nodes['Distribute'].inputs['Seed'].default_value = seed
            bpy.data.lights['Sun'].energy = intensity
            
            # Render
            bpy.context.scene.render.filepath = f"/output/d{density}_s{seed}_i{intensity}"
            bpy.ops.render.render(write_still=True)
            
            # Save metadata
            metadata = {'density': density, 'seed': seed, 'intensity': intensity}
            with open(f"/output/d{density}_s{seed}_i{intensity}.json", 'w') as f:
                json.dump(metadata, f)
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
