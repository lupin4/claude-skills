---
name: "fortran-lapack-blas"
description: "Use when linking or calling LAPACK/BLAS routines. Covers linking flags, vendor implementations (MKL, OpenBLAS, Accelerate), interface blocks, workspace queries, and batch operations."
---

# Fortran LAPACK/BLAS

Linking and using LAPACK/BLAS from Fortran kernels. Covers vendor-specific implementations, explicit interface blocks, workspace query patterns, and performance tuning.

---

## Overview

### What This Skill Does

LAPACK (Linear Algebra PACKage) and BLAS (Basic Linear Algebra Subprograms) are the foundation of numerical linear algebra. This skill covers how to link, call, and optimize these routines in the forKernels ecosystem.

---

## Linking

### Flags by Platform

| Platform | Library | Link Flags |
|----------|---------|-----------|
| Linux (reference) | Netlib LAPACK | `-llapack -lblas` |
| Linux (OpenBLAS) | OpenBLAS | `-lopenblas` |
| Linux (MKL) | Intel MKL | `-lmkl_gf_lp64 -lmkl_sequential -lmkl_core -lpthread -lm` |
| macOS | Accelerate | `-framework Accelerate` |
| Zig build | via addSystemLibrary | `lib.linkSystemLibrary("lapack")` |

### build.zig Pattern

```zig
lib.linkSystemLibrary("lapack");
lib.linkSystemLibrary("blas");
lib.linkSystemLibrary("gfortran");
lib.linkSystemLibrary("m");
```

---

## Key LAPACK Routines

### Linear Systems (Ax = b)

```fortran
! DGESV: General system via LU factorization
subroutine fm_solve_linear(a, b, n, nrhs, status) bind(C, name="fm_solve_linear")
  integer(c_int), intent(in), value :: n, nrhs
  real(c_double), intent(inout) :: a(n, n)  ! Overwritten with LU
  real(c_double), intent(inout) :: b(n, nrhs)  ! Overwritten with solution
  integer(c_int), intent(out) :: status
  integer :: ipiv(n), info
  call dgesv(n, nrhs, a, n, ipiv, b, n, info)
  status = info
end subroutine
```

### Eigenvalues

```fortran
! DSYEV: Symmetric eigenvalue decomposition
subroutine fm_eigen_symmetric(a, w, n, status) bind(C, name="fm_eigen_symmetric")
  integer(c_int), intent(in), value :: n
  real(c_double), intent(inout) :: a(n, n)  ! Overwritten with eigenvectors
  real(c_double), intent(out) :: w(n)  ! Eigenvalues ascending
  integer(c_int), intent(out) :: status
  real(c_double) :: work(1)
  real(c_double), allocatable :: work_full(:)
  integer :: lwork, info

  ! Workspace query
  lwork = -1
  call dsyev('V', 'U', n, a, n, w, work, lwork, info)
  lwork = int(work(1))
  allocate(work_full(lwork))
  call dsyev('V', 'U', n, a, n, w, work_full, lwork, info)
  deallocate(work_full)
  status = info
end subroutine
```

### SVD

```fortran
! DGESVD: Singular Value Decomposition
! A = U * S * V^T
```

---

## Workspace Query Pattern

LAPACK routines that need workspace follow a two-call pattern:

```fortran
! Step 1: Query optimal workspace (lwork = -1)
lwork = -1
call dsyev('V', 'U', n, a, n, w, work_query, lwork, info)
lwork = int(work_query(1))

! Step 2: Allocate and compute
allocate(work(lwork))
call dsyev('V', 'U', n, a, n, w, work, lwork, info)
deallocate(work)
```

---

## BLAS Levels

| Level | Operations | Key Routines |
|-------|-----------|-------------|
| 1 | Vector-vector | `dscal`, `dcopy`, `ddot`, `dnrm2`, `daxpy` |
| 2 | Matrix-vector | `dgemv`, `dsymv`, `dtrsv` |
| 3 | Matrix-matrix | `dgemm`, `dsyrk`, `dtrsm` |

### DGEMM (Matrix Multiply)

```fortran
! C = alpha * A * B + beta * C
call dgemm('N', 'N', m, n, k, 1.0d0, a, lda, b, ldb, 0.0d0, c, ldc)
```

---

## Anti-Patterns

- **Not querying workspace** — hardcoded workspace sizes cause crashes or suboptimal performance
- **Wrong leading dimension** — lda must be >= max(1, m), not n
- **Ignoring INFO return** — always check: info < 0 means bad argument, info > 0 means failure
- **Using reference BLAS in production** — use vendor-optimized (OpenBLAS, MKL, Accelerate)
- **Assuming row-major** — LAPACK is column-major, always

---

## Cross-References

- [fortran-module-design](../fortran-module-design/SKILL.md) — Interface blocks
- [zortran-bind-c](../../zortran/zortran-bind-c/SKILL.md) — Wrapping LAPACK for Zig
- [fortran-lie-groups](../fortran-lie-groups/SKILL.md) — Uses BLAS for matrix ops
