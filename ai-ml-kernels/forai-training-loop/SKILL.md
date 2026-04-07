---
name: "forai-training-loop"
description: "Use when implementing Training loop infrastructure, gradient accumulation, mixed precision, KV cache layout, RL reward integration."
---

# forAI Training Loop

Training loop infrastructure, gradient accumulation, mixed precision, KV cache layout, RL reward integration. These are Fortran+Zig implementations of ML primitives — NOT Python/PyTorch wrappers.

---

## Overview

### What This Skill Does

Implements forAI Training Loop as numerical kernels in Fortran with Zig orchestration. Part of the forKernels ML ecosystem.

## Training Loop Skeleton

```fortran
! Single training step: forward → loss → backward → update
subroutine fai_train_step(model, input, target, lr, loss, status) &
    bind(C, name="fai_train_step")
  type(c_ptr), intent(inout) :: model
  real(c_double), intent(in) :: input(*), target(*)
  real(c_double), intent(in), value :: lr
  real(c_double), intent(out) :: loss
  integer(c_int), intent(out) :: status
  ! 1. Forward pass
  ! 2. Compute loss
  ! 3. Backward pass (accumulate gradients)
  ! 4. Update parameters: θ = θ - lr * ∇θ
end subroutine
```

## KV Cache Layout

```fortran
! KV cache for transformer inference
type, bind(C) :: ft_kv_cache_t
  real(c_double), pointer :: k_cache(:,:,:)  ! (head_dim, n_heads, seq_len)
  real(c_double), pointer :: v_cache(:,:,:)  ! (head_dim, n_heads, seq_len)
  integer(c_int) :: current_pos              ! Current position in cache
  integer(c_int) :: max_seq_len
end type

! Column-major: head_dim is contiguous (fast BLAS access)
```

## Gradient Accumulation

```fortran
! Accumulate gradients over micro-batches before update
subroutine fai_accumulate_grad(grad_acc, grad_new, n, step, accum_steps) &
    bind(C, name="fai_accumulate_grad")
  integer(c_int), intent(in), value :: n, step, accum_steps
  real(c_double), intent(inout) :: grad_acc(n)
  real(c_double), intent(in) :: grad_new(n)
  ! Add gradient
  call daxpy(n, 1.0d0, grad_new, 1, grad_acc, 1)
  ! If accumulation complete, scale by 1/accum_steps
  if (mod(step, accum_steps) == 0) then
    call dscal(n, 1.0d0/dble(accum_steps), grad_acc, 1)
  end if
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
