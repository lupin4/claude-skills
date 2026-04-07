---
name: "fortran-module-design"
description: "Use when organizing Fortran modules. Covers USE/ONLY discipline, IMPLICIT NONE, public/private visibility, submodule patterns, abstract interfaces, and one-module-per-file convention."
---

# Fortran Module Design

Module organization patterns for production Fortran code. Covers visibility control, import discipline, submodules for implementation hiding, and naming conventions.

---

## Overview

### Core Rules

1. **IMPLICIT NONE everywhere** — non-negotiable
2. **PRIVATE by default** — only export what's needed
3. **USE, ONLY** — never bare `USE module` (pollutes namespace)
4. **One module per file** — file name matches module name
5. **snake_case** — all identifiers

---

## Module Template

```fortran
module fm_rigid_body
  use iso_c_binding, only: c_int, c_double, c_ptr
  use fm_math_types, only: ft_vec3_t, ft_quat_t
  implicit none
  private

  ! Public API
  public :: fm_rigid_body_step
  public :: fm_rigid_body_init
  public :: ft_rigid_body_t

  type, bind(C) :: ft_rigid_body_t
    type(ft_vec3_t) :: position
    type(ft_quat_t) :: orientation
    type(ft_vec3_t) :: velocity
    type(ft_vec3_t) :: angular_velocity
    real(c_double)  :: mass
  end type

contains

  subroutine fm_rigid_body_init(body, mass) bind(C, name="fm_rigid_body_init")
    type(ft_rigid_body_t), intent(out) :: body
    real(c_double), intent(in), value :: mass
    body%mass = mass
    body%position = ft_vec3_t(0.0d0, 0.0d0, 0.0d0)
    body%orientation = ft_quat_t(1.0d0, 0.0d0, 0.0d0, 0.0d0)
    body%velocity = ft_vec3_t(0.0d0, 0.0d0, 0.0d0)
    body%angular_velocity = ft_vec3_t(0.0d0, 0.0d0, 0.0d0)
  end subroutine

  subroutine fm_rigid_body_step(body, dt, status) bind(C, name="fm_rigid_body_step")
    type(ft_rigid_body_t), intent(inout) :: body
    real(c_double), intent(in), value :: dt
    integer(c_int), intent(out) :: status
    ! Semi-implicit Euler integration
    body%position%x = body%position%x + body%velocity%x * dt
    body%position%y = body%position%y + body%velocity%y * dt
    body%position%z = body%position%z + body%velocity%z * dt
    status = 0
  end subroutine

end module fm_rigid_body
```

---

## Submodule Pattern

```fortran
! fm_solver.f90 — interface only
module fm_solver
  implicit none
  private
  public :: fm_solve_linear

  interface
    module subroutine fm_solve_linear(a, b, n, status)
      real(8), intent(inout) :: a(n,n), b(n)
      integer, intent(in) :: n
      integer, intent(out) :: status
    end subroutine
  end interface
end module

! fm_solver_impl.f90 — implementation
submodule (fm_solver) fm_solver_impl
contains
  module subroutine fm_solve_linear(a, b, n, status)
    real(8), intent(inout) :: a(n,n), b(n)
    integer, intent(in) :: n
    integer, intent(out) :: status
    ! Implementation here — changes don't recompile dependents
  end subroutine
end submodule
```

---

## Anti-Patterns

- **Bare USE** — `use my_module` imports everything; always use `only:`
- **PUBLIC by default** — set `private` at module level, selectively export
- **Implicit typing** — `implicit none` in every scope, always
- **Multiple modules per file** — one module = one file = one compilation unit
- **CamelCase** — Fortran is case-insensitive; snake_case is the convention

---

## Cross-References

- [zortran-bind-c](../../zortran/zortran-bind-c/SKILL.md) — bind(C) exports from modules
- [fortran-burkhardt](../fortran-burkhardt/SKILL.md) — Wrapping legacy into modules
