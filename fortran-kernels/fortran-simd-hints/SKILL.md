---
name: "fortran-simd-hints"
description: "Use when optimizing Fortran loops for SIMD vectorization. Covers DO CONCURRENT, !$omp simd, compiler directives, vectorization reports, and GPU offload with OpenMP target."
---

# Fortran SIMD Hints

Loop vectorization directives and patterns for Fortran numerical kernels. Covers DO CONCURRENT, OpenMP SIMD, compiler-specific directives, and GPU offload.

---

## Overview

### What This Skill Does

Guides the use of compiler directives and language features to ensure Fortran loops vectorize effectively on modern CPUs and optionally offload to GPUs.

---

## DO CONCURRENT

```fortran
! Simple vectorizable loop
do concurrent (i = 1:n)
  y(i) = a * x(i) + y(i)
end do

! 2D
do concurrent (i = 1:m, j = 1:n)
  c(i,j) = a(i,j) + b(i,j)
end do

! With mask
do concurrent (i = 1:n, x(i) > 0.0d0)
  y(i) = sqrt(x(i))
end do
```

**Limitations:** No dependencies between iterations, no EXIT/CYCLE, no I/O, no ALLOCATE.

---

## OpenMP SIMD

```fortran
!$omp simd
do i = 1, n
  y(i) = a * x(i) + y(i)
end do

!$omp simd reduction(+:total)
do i = 1, n
  total = total + x(i) * y(i)
end do

! Combined parallel + SIMD
!$omp parallel do simd
do i = 1, n
  z(i) = compute_expensive(x(i), y(i))
end do
```

---

## OpenMP Target (GPU Offload)

```fortran
!$omp target teams distribute parallel do simd map(to:x,y) map(from:z)
do i = 1, n
  z(i) = x(i) * y(i)
end do
!$omp end target teams distribute parallel do simd
```

---

## Vectorization Reports

```bash
# GFortran
gfortran -O3 -fopt-info-vec-optimized -fopt-info-vec-missed

# Intel
ifort -O3 -qopt-report=5 -qopt-report-phase=vec

# LLVM/Flang
flang -O3 -Rpass=loop-vectorize -Rpass-missed=loop-vectorize
```

---

## Common Vectorization Blockers

| Blocker | Fix |
|---------|-----|
| Loop-carried dependency | Restructure algorithm or use reduction |
| Function call in loop body | Inline or add `!$omp declare simd` |
| Non-contiguous array access | Use contiguous attribute or reshape |
| Conditional branch | Use merge() intrinsic or mask |
| Pointer/allocatable aliasing | Add `contiguous` attribute |

---

## Anti-Patterns

- **DO CONCURRENT with side effects** — no I/O, no global state mutation
- **Ignoring vectorization reports** — always check what actually vectorized
- **Premature GPU offload** — verify CPU vectorization first, GPU only for large parallel work
- **Assuming all loops vectorize** — check with `-fopt-info-vec-missed`

---

## Cross-References

- [fortran-module-design](../fortran-module-design/SKILL.md) — Module structure affects inlining
- [fortran-lapack-blas](../fortran-lapack-blas/SKILL.md) — BLAS is already vectorized, don't re-vectorize
