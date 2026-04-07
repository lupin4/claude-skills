---
name: "fortorch-tensor-model"
description: "Use when implementing View-based refcounted tensor memory model, RSSM state transitions, VQ codebook management in Fortran+Zig."
---

# forTorch Tensor Model

View-based refcounted tensor memory model, RSSM state transitions, VQ codebook management in Fortran+Zig. These are Fortran+Zig implementations of ML primitives — NOT Python/PyTorch wrappers.

---

## Overview

### What This Skill Does

Implements forTorch Tensor Model as numerical kernels in Fortran with Zig orchestration. Part of the forKernels ML ecosystem.

## Tensor Memory Model

### View-Based Architecture

```fortran
type, bind(C) :: ft_tensor_t
  type(c_ptr) :: data          ! Pointer to raw f64 array
  integer(c_int) :: ndim       ! Number of dimensions
  integer(c_int) :: shape(8)   ! Shape (max 8 dims)
  integer(c_int) :: strides(8) ! Strides in elements
  integer(c_int) :: offset     ! Offset into data buffer
  integer(c_int) :: refcount   ! Reference count
  logical(c_bool) :: owns_data ! Whether this tensor owns the buffer
end type
```

### View Semantics (No Copy)

```fortran
! Create a view (shared memory, no copy)
subroutine fm_tensor_view(src, dst, dim, start, length) bind(C, name="fm_tensor_view")
  type(ft_tensor_t), intent(in) :: src
  type(ft_tensor_t), intent(out) :: dst
  integer(c_int), intent(in), value :: dim, start, length
  ! dst shares src's data buffer
  dst%data = src%data
  dst%owns_data = .false.
  dst%refcount = 1  ! View has its own refcount
  ! Adjust offset and shape for the slice
  dst%offset = src%offset + start * src%strides(dim)
  dst%shape(dim) = length
end subroutine
```

### RSSM (Recurrent State Space Model)

State transition kernel: h_t = f(h_{t-1}, x_t)

```fortran
subroutine fml_rssm_step(h_prev, x, h_next, hidden_dim, input_dim, status) &
    bind(C, name="fml_rssm_step")
  integer(c_int), intent(in), value :: hidden_dim, input_dim
  real(c_double), intent(in) :: h_prev(hidden_dim), x(input_dim)
  real(c_double), intent(out) :: h_next(hidden_dim)
  integer(c_int), intent(out) :: status
  ! GRU-like gating: z = σ(W_z·[h,x]), r = σ(W_r·[h,x])
  ! h_next = (1-z)⊙h + z⊙tanh(W·[r⊙h, x])
end subroutine
```

### Vector Quantization (VQ)

```fortran
! Find nearest codebook entry
subroutine fml_vq_encode(x, codebook, n_codes, dim, index, status) &
    bind(C, name="fml_vq_encode")
  integer(c_int), intent(in), value :: n_codes, dim
  real(c_double), intent(in) :: x(dim), codebook(dim, n_codes)
  integer(c_int), intent(out) :: index, status
  ! Argmin ||x - codebook(:,k)||^2 over k
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
