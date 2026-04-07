---
name: "hopfield-attention"
description: "Use when implementing Classical Hopfield energy, modern Hopfield (exponential capacity), equivalence to softmax attention, energy-based retrieval."
---

# Hopfield-Attention Equivalence

Classical Hopfield energy, modern Hopfield (exponential capacity), equivalence to softmax attention, energy-based retrieval. These are Fortran+Zig implementations of ML primitives — NOT Python/PyTorch wrappers.

---

## Overview

### What This Skill Does

Implements Hopfield-Attention Equivalence as numerical kernels in Fortran with Zig orchestration. Part of the forKernels ML ecosystem.

## Classical Hopfield Network

Energy function: E = -½ Σᵢⱼ wᵢⱼ sᵢ sⱼ

Update rule: sᵢ = sign(Σⱼ wᵢⱼ sⱼ)

Storage capacity: ~0.138N patterns (for N neurons)

## Modern Hopfield Network (Ramsauer et al., 2020)

Energy function: E = -log Σᵢ exp(βξᵢᵀx) + ½x²  + const

Update rule: x_new = Ξ · softmax(βΞᵀx)

**Exponential storage capacity:** exp(βd/2) patterns

## Equivalence to Softmax Attention

```
Hopfield retrieval:  x_new = Ξ · softmax(β · Ξᵀ · x)
Attention:           output = V · softmax(K^T · Q / √d_k)

Mapping:
  Ξ (stored patterns) ↔ V (values)
  Ξ (keys)            ↔ K (keys)
  x (query state)     ↔ Q (query)
  β                   ↔ 1/√d_k
```

### Fortran Kernel

```fortran
! Hopfield retrieval = attention computation
subroutine fml_hopfield_retrieve(query, keys, values, n_patterns, dim, beta, output) &
    bind(C, name="fml_hopfield_retrieve")
  integer(c_int), intent(in), value :: n_patterns, dim
  real(c_double), intent(in), value :: beta
  real(c_double), intent(in) :: query(dim), keys(dim, n_patterns), values(dim, n_patterns)
  real(c_double), intent(out) :: output(dim)
  real(c_double) :: logits(n_patterns), weights(n_patterns), max_logit
  integer :: i
  ! logits = β · keys^T · query
  call dgemv('T', dim, n_patterns, beta, keys, dim, query, 1, 0.0d0, logits, 1)
  ! Stable softmax
  max_logit = maxval(logits)
  weights = exp(logits - max_logit)
  weights = weights / sum(weights)
  ! output = values · weights
  call dgemv('N', dim, n_patterns, 1.0d0, values, dim, weights, 1, 0.0d0, output, 1)
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
