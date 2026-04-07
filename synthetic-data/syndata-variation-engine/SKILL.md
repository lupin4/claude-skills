---
name: "syndata-variation-engine"
description: "Use when Parameter sweep design, Latin hypercube vs grid vs random sampling, seed management, variation taxonomy (geometric, photometric, semantic)."
---

# Synthetic Data Variation Engine

Parameter sweep design, Latin hypercube vs grid vs random sampling, seed management, variation taxonomy (geometric, photometric, semantic).

---

## Overview

### What This Skill Does

Part of the robotics synthetic data pipeline. Generates training data using headless Blender for robot manipulation tasks targeting NVIDIA Newton simulation.

## Variation Taxonomy

| Category | Parameters | Range |
|----------|-----------|-------|
| **Geometric** | Object pose (x,y,z,rx,ry,rz) | Task-dependent |
| **Geometric** | Robot start configuration | Joint limits |
| **Photometric** | Light intensity | 100-5000 lux |
| **Photometric** | Light color temperature | 3000-6500 K |
| **Photometric** | HDR environment map | Library selection |
| **Material** | Object texture | Texture library |
| **Material** | Surface roughness | 0.1-0.9 |
| **Semantic** | Object type substitution | Same grasp category |
| **Camera** | Viewpoint (position, look-at) | Hemisphere sampling |

## Sampling Strategies

| Strategy | Best For | Coverage |
|----------|---------|----------|
| Grid | Low-dim (≤3 params) | Uniform but exponential cost |
| Random | Baseline, any dim | Gaps possible |
| Latin Hypercube (LHS) | Medium-dim (4-20 params) | Even marginal coverage |
| Sobol | High-dim (>20 params) | Low-discrepancy, best coverage |

## Seed Management

```python
import numpy as np

def generate_seeds(base_seed: int, n_variations: int) -> list[int]:
    rng = np.random.default_rng(base_seed)
    return rng.integers(0, 2**31, size=n_variations).tolist()

# Every randomization call uses its own sub-seed
pose_rng = np.random.default_rng(seeds[0])
light_rng = np.random.default_rng(seeds[1])
texture_rng = np.random.default_rng(seeds[2])
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
