---
name: "syndata-dataset-curator"
description: "Use when Output schema, annotation formats (COCO, PASCAL VOC, custom JSON), dataset versioning (git-lfs, DVC), quality filtering, split generation."
---

# Synthetic Data Dataset Curator

Output schema, annotation formats (COCO, PASCAL VOC, custom JSON), dataset versioning (git-lfs, DVC), quality filtering, split generation.

---

## Overview

### What This Skill Does

Part of the robotics synthetic data pipeline. Generates training data using headless Blender for robot manipulation tasks targeting NVIDIA Newton simulation.

## Output Directory Structure

```
dataset_v1/
├── images/
│   ├── rgb/           # Color images (PNG)
│   ├── depth/         # Depth maps (EXR, 32-bit float)
│   ├── normals/       # Surface normals (EXR)
│   └── segmentation/  # Instance segmentation masks (PNG, 16-bit)
├── annotations/
│   ├── coco.json      # COCO format annotations
│   └── metadata.json  # Per-frame render parameters
├── splits/
│   ├── train.txt
│   ├── val.txt
│   └── test.txt
└── README.md          # Dataset card
```

## COCO Annotation Schema

```json
{
  "images": [{"id": 1, "file_name": "rgb/000001.png", "width": 1280, "height": 720}],
  "annotations": [{"id": 1, "image_id": 1, "category_id": 1, "bbox": [x,y,w,h], "segmentation": [...]}],
  "categories": [{"id": 1, "name": "mug", "supercategory": "object"}]
}
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
