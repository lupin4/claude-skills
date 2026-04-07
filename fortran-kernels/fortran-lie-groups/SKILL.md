---
name: "fortran-lie-groups"
description: "Use when implementing SO(3)/SE(3), quaternion, or dual quaternion kernels for robotics. Covers rotation representations, Hamilton quaternion convention, exponential/logarithmic maps, and Jacobian computation."
---

# Fortran Lie Groups

SO(3)/SE(3), quaternion, and dual quaternion kernel patterns for robotics and physics simulation. All rotation math uses Hamilton quaternion convention (w, x, y, z ordering).

---

## Overview

### What This Skill Does

Provides Fortran kernel patterns for 3D rotation and rigid body transformation math — the foundation of robotics kinematics and physics simulation.

---

## Quaternion Convention

**Hamilton convention: q = (w, x, y, z) where w is the scalar part.**

```fortran
type, bind(C) :: ft_quat_t
  real(c_double) :: w, x, y, z  ! scalar first
end type
```

### Quaternion Multiplication

```fortran
subroutine fm_quat_mul(a, b, c) bind(C, name="fm_quat_mul")
  type(ft_quat_t), intent(in) :: a, b
  type(ft_quat_t), intent(out) :: c
  c%w = a%w*b%w - a%x*b%x - a%y*b%y - a%z*b%z
  c%x = a%w*b%x + a%x*b%w + a%y*b%z - a%z*b%y
  c%y = a%w*b%y - a%x*b%z + a%y*b%w + a%z*b%x
  c%z = a%w*b%z + a%x*b%y - a%y*b%x + a%z*b%w
end subroutine
```

### Quaternion to Rotation Matrix

```fortran
subroutine fm_quat_to_rotmat(q, R) bind(C, name="fm_quat_to_rotmat")
  type(ft_quat_t), intent(in) :: q
  real(c_double), intent(out) :: R(3,3)
  real(c_double) :: xx, yy, zz, xy, xz, yz, wx, wy, wz

  xx = q%x*q%x; yy = q%y*q%y; zz = q%z*q%z
  xy = q%x*q%y; xz = q%x*q%z; yz = q%y*q%z
  wx = q%w*q%x; wy = q%w*q%y; wz = q%w*q%z

  R(1,1) = 1.0d0 - 2.0d0*(yy + zz)
  R(2,1) = 2.0d0*(xy + wz)
  R(3,1) = 2.0d0*(xz - wy)
  R(1,2) = 2.0d0*(xy - wz)
  R(2,2) = 1.0d0 - 2.0d0*(xx + zz)
  R(3,2) = 2.0d0*(yz + wx)
  R(1,3) = 2.0d0*(xz + wy)
  R(2,3) = 2.0d0*(yz - wx)
  R(3,3) = 1.0d0 - 2.0d0*(xx + yy)
end subroutine
```

---

## Exponential and Logarithmic Maps

### so(3) → SO(3) (Rodrigues)

```fortran
! Exponential map: axis-angle vector → rotation matrix
! omega = theta * axis (3-vector)
subroutine fm_so3_exp(omega, R) bind(C, name="fm_so3_exp")
  real(c_double), intent(in) :: omega(3)
  real(c_double), intent(out) :: R(3,3)
  real(c_double) :: theta, K(3,3), I3(3,3)

  theta = sqrt(omega(1)**2 + omega(2)**2 + omega(3)**2)
  if (theta < 1.0d-10) then
    R = reshape([1,0,0, 0,1,0, 0,0,1], [3,3])
    return
  end if
  ! K = skew-symmetric matrix of omega/theta
  call skew3(omega/theta, K)
  I3 = reshape([1,0,0, 0,1,0, 0,0,1], [3,3])
  ! Rodrigues: R = I + sin(θ)K + (1-cos(θ))K²
  R = I3 + sin(theta)*K + (1.0d0 - cos(theta))*matmul(K, K)
end subroutine
```

### SE(3) Representation

```fortran
type, bind(C) :: ft_pose_t
  real(c_double) :: rotation(4)     ! quaternion (w,x,y,z)
  real(c_double) :: translation(3)  ! (x,y,z)
end type
```

---

## Dual Quaternions

```fortran
type, bind(C) :: ft_dualquat_t
  type(ft_quat_t) :: real_part   ! rotation
  type(ft_quat_t) :: dual_part   ! translation encoded
end type

! Construct from rotation quaternion + translation vector
subroutine fm_dualquat_from_rt(q, t, dq) bind(C, name="fm_dualquat_from_rt")
  type(ft_quat_t), intent(in) :: q
  real(c_double), intent(in) :: t(3)
  type(ft_dualquat_t), intent(out) :: dq
  type(ft_quat_t) :: t_quat

  t_quat%w = 0.0d0
  t_quat%x = t(1); t_quat%y = t(2); t_quat%z = t(3)

  dq%real_part = q
  ! dual = 0.5 * t_quat * q
  call fm_quat_mul(t_quat, q, dq%dual_part)
  dq%dual_part%w = 0.5d0 * dq%dual_part%w
  dq%dual_part%x = 0.5d0 * dq%dual_part%x
  dq%dual_part%y = 0.5d0 * dq%dual_part%y
  dq%dual_part%z = 0.5d0 * dq%dual_part%z
end subroutine
```

---

## Anti-Patterns

- **JPL vs Hamilton confusion** — always use Hamilton (w,x,y,z), document everywhere
- **Non-unit quaternions** — normalize after every multiply chain
- **Gimbal lock with Euler angles** — use quaternions for interpolation, Euler only for display
- **Transposed rotation matrix** — column-major Fortran vs row-major C: verify with known rotations

---

## Cross-References

- [fortran-lapack-blas](../fortran-lapack-blas/SKILL.md) — BLAS for matrix ops in rotation math
- [physics-sim](../../physics-sim/) — Rigid body dynamics uses SE(3)
- [synthetic-data](../../synthetic-data/) — Pose randomization uses SO(3)
