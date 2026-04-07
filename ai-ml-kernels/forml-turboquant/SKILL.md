---
name: "forml-turboquant"
description: "Use when implementing PolarQuant + QJL extreme quantization (arxiv 2504.19874), polar decomposition, 1-bit/2-bit kernels."
---

# forML TurboQuant

PolarQuant + QJL extreme quantization (arxiv 2504.19874), polar decomposition, 1-bit/2-bit kernels. These are Fortran+Zig implementations of ML primitives — NOT Python/PyTorch wrappers.

---

## Overview

### What This Skill Does

Implements forML TurboQuant as numerical kernels in Fortran with Zig orchestration. Part of the forKernels ML ecosystem.

## TurboQuant Algorithm (arxiv 2504.19874)

### PolarQuant: Polar Decomposition of Weight Matrices

W = U·P where U is unitary, P is positive semi-definite.

Quantize P (which is better conditioned) instead of W directly.

```fortran
! Step 1: Polar decomposition W = U·P
! Using SVD: W = U_svd · S · V^T → U = U_svd · V^T, P = V · S · V^T
subroutine fml_polar_decompose(w, u, p, m, n, status) &
    bind(C, name="fml_polar_decompose")
  integer(c_int), intent(in), value :: m, n
  real(c_double), intent(in) :: w(m, n)
  real(c_double), intent(out) :: u(m, n), p(n, n)
  integer(c_int), intent(out) :: status
  ! Uses LAPACK DGESVD internally
end subroutine

! Step 2: Quantize P to low-bit (1-bit or 2-bit)
subroutine fml_quantize_2bit(matrix, quantized, scale, n, status) &
    bind(C, name="fml_quantize_2bit")
  integer(c_int), intent(in), value :: n
  real(c_double), intent(in) :: matrix(n, n)
  integer(c_int8_t), intent(out) :: quantized(n, n)  ! 2-bit packed
  real(c_double), intent(out) :: scale
  integer(c_int), intent(out) :: status
end subroutine
```

### QJL: Quantized Johnson-Lindenstrauss

Random projection for dimensionality reduction, then quantize the projected space.

```fortran
! JL projection: y = (1/√k) · R · x  where R is random ±1
subroutine fml_qjl_project(x, y, n, k, seed, status) &
    bind(C, name="fml_qjl_project")
  integer(c_int), intent(in), value :: n, k, seed
  real(c_double), intent(in) :: x(n)
  real(c_double), intent(out) :: y(k)  ! k << n
  integer(c_int), intent(out) :: status
end subroutine
```

---

## Anti-Patterns

- **Using Python/PyTorch for kernels** — this is Fortran+Zig, not Python
- **Row-major assumptions** — Fortran is column-major, design layouts accordingly
- **Skipping BLAS** — use DGEMM/DGEMV for matrix ops, don't hand-roll

---

## Cross-References

- [fortran-kernels/fortran-lapack-blas](../../fortran-kernels/fortran-lapack-blas/SKILL.md) — BLAS/LAPACK for matrix ops
- [zortran/](../../zortran/) — Fortran↔Zig interop
