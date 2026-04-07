---
name: "forbayes-gp"
description: "Use when implementing Gaussian process surrogates, kernel functions (RBF, Matern), acquisition functions (EI, UCB), MCMC, Pareto dominance."
---

# forBayes Gaussian Process

Gaussian process surrogates, kernel functions (RBF, Matern), acquisition functions (EI, UCB), MCMC, Pareto dominance. These are Fortran+Zig implementations of ML primitives — NOT Python/PyTorch wrappers.

---

## Overview

### What This Skill Does

Implements forBayes Gaussian Process as numerical kernels in Fortran with Zig orchestration. Part of the forKernels ML ecosystem.

## Gaussian Process Kernels

### RBF (Squared Exponential)

k(x, x') = σ² · exp(-||x - x'||² / (2·l²))

```fortran
subroutine fm_gp_rbf_kernel(x1, x2, n1, n2, dim, sigma2, length, K) &
    bind(C, name="fm_gp_rbf_kernel")
  integer(c_int), intent(in), value :: n1, n2, dim
  real(c_double), intent(in), value :: sigma2, length
  real(c_double), intent(in) :: x1(dim, n1), x2(dim, n2)
  real(c_double), intent(out) :: K(n1, n2)
  integer :: i, j
  real(c_double) :: dist2
  do j = 1, n2
    do i = 1, n1
      dist2 = sum((x1(:,i) - x2(:,j))**2)
      K(i,j) = sigma2 * exp(-dist2 / (2.0d0 * length**2))
    end do
  end do
end subroutine
```

### GP Inference via Cholesky

```fortran
! K = K(X,X) + σ²_noise · I
! L = cholesky(K)
! α = L^T \ (L \ y)
! μ* = K(X*,X) · α
! Solve with LAPACK DPOTRF + DPOTRS
```

### Acquisition Functions

| Function | Formula | Use When |
|----------|---------|----------|
| EI (Expected Improvement) | E[max(f(x) - f_best, 0)] | Default choice |
| UCB (Upper Confidence Bound) | μ(x) + κ·σ(x) | Exploration-heavy |
| PI (Probability of Improvement) | P(f(x) > f_best) | Exploitation-heavy |

---

## Anti-Patterns

- **Using Python/PyTorch for kernels** — this is Fortran+Zig, not Python
- **Row-major assumptions** — Fortran is column-major, design layouts accordingly
- **Skipping BLAS** — use DGEMM/DGEMV for matrix ops, don't hand-roll

---

## Cross-References

- [fortran-kernels/fortran-lapack-blas](../../fortran-kernels/fortran-lapack-blas/SKILL.md) — BLAS/LAPACK for matrix ops
- [zortran/](../../zortran/) — Fortran↔Zig interop
