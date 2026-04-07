# Lie Group Formulas Reference

## SO(3) — 3D Rotation Group

**Dimension:** 3 parameters, 3×3 orthogonal matrices with det=+1

### Representations
| Representation | Parameters | Singularity-Free | Compact |
|---------------|-----------|------------------|---------|
| Rotation matrix | 9 (3 constraints) | Yes | No |
| Quaternion | 4 (1 constraint) | Yes | Yes |
| Euler angles | 3 | No (gimbal lock) | Yes |
| Axis-angle | 3 | No (at θ=0) | Yes |
| Rodrigues vector | 3 | No (at π) | Yes |

### Key Formulas

**Rodrigues formula:** R = I + sin(θ)[ω]× + (1-cos(θ))[ω]×²

**Quaternion to axis-angle:** θ = 2·arccos(w), axis = (x,y,z)/sin(θ/2)

**Quaternion SLERP:** q(t) = q₁·sin((1-t)Ω)/sin(Ω) + q₂·sin(tΩ)/sin(Ω)

where Ω = arccos(q₁·q₂)

## SE(3) — Rigid Body Transformation Group

**Dimension:** 6 parameters (3 rotation + 3 translation)

**Homogeneous matrix:**
```
T = | R  t |    R ∈ SO(3), t ∈ R³
    | 0  1 |
```

**Twist (se(3) algebra element):**
```
ξ = | ω |    ω = angular velocity
    | v |    v = linear velocity
```

**Exponential map (twist → transform):**
```
exp(ξθ) = | exp([ω]×θ)   Vt |
          | 0            1  |

V = I + (1-cos(θ))/θ² · [ω]× + (θ-sin(θ))/θ³ · [ω]×²
```

## Quaternion Algebra

**Multiplication (Hamilton):** q₁q₂ = (w₁w₂ - v₁·v₂, w₁v₂ + w₂v₁ + v₁×v₂)

**Conjugate:** q* = (w, -x, -y, -z)

**Inverse:** q⁻¹ = q*/|q|²

**Rotation of point p:** p' = q p q*  (where p = (0, px, py, pz))

**Composition:** R₁R₂ ↔ q₁q₂

## Dual Quaternion

**q̂ = q_r + ε·q_d** where ε² = 0

**From (R, t):** q_r = rotation quaternion, q_d = ½·t_quat·q_r

**To (R, t):** R = quat_to_mat(q_r), t = 2·q_d·q_r*
