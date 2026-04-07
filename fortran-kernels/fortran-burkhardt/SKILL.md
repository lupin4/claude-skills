---
name: "fortran-burkhardt"
description: "Use when integrating John Burkardt's numerical routines into modern Fortran modules. Covers module wrapping, USE/ONLY imports, naming conventions, and modernization patterns."
---

# Fortran Burkhardt

Patterns for integrating burkardt-modern routines — John Burkardt's extensive numerical library modernized to F2008+. Covers wrapping legacy routines into clean modules with proper interfaces.

---

## Table of Contents

- [Overview](#overview)
- [Module Wrapping Strategy](#module-wrapping-strategy)
- [Routine Categories](#routine-categories)
- [Modernization Patterns](#modernization-patterns)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

John Burkardt's library contains 1000+ numerical routines covering quadrature, interpolation, random number generation, combinatorics, geometry, and more. This skill defines how to wrap these routines into clean Fortran modules for use in the forKernels ecosystem.

### Prerequisites

- burkardt-modern source (F90/F2008 versions)
- Understanding of the routine's mathematical purpose

---

## Module Wrapping Strategy

### One Module per Routine Family

```fortran
module fm_quadrature
  use iso_c_binding
  implicit none
  private

  public :: fm_gauss_legendre, fm_gauss_hermite, fm_simpson

contains

  subroutine fm_gauss_legendre(n, x, w) bind(C, name="fm_gauss_legendre")
    integer(c_int), intent(in), value :: n
    real(c_double), intent(out) :: x(n), w(n)
    ! Wraps burkardt's legendre_compute
    call legendre_compute(n, x, w)
  end subroutine

  ! Internal burkardt routine (not exported)
  subroutine legendre_compute(n, x, w)
    integer, intent(in) :: n
    real(8), intent(out) :: x(n), w(n)
    ! ... burkardt implementation ...
  end subroutine

end module fm_quadrature
```

### Naming Convention

```
Original burkardt:  r8_gamma_log(x)
Wrapped module:     fm_special_functions
Wrapped function:   fm_log_gamma(x)  bind(C, name="fm_log_gamma")
```

**Rules:**
- Module name: `fm_{category}` (e.g., `fm_quadrature`, `fm_interpolation`)
- Function name: `fm_{descriptive_name}` — may differ from burkardt's original
- Internal burkardt routines stay private, not exported via bind(C)

---

## Routine Categories

| Category | Key Routines | Module |
|----------|-------------|--------|
| Quadrature | gauss_legendre, gauss_hermite, clenshaw_curtis | fm_quadrature |
| Interpolation | lagrange, spline_cubic, rbf | fm_interpolation |
| Random | halton, sobol, uniform, normal | fm_random |
| Geometry | polygon_area, convex_hull, voronoi | fm_geometry |
| Special Functions | gamma, beta, bessel, erf | fm_special_functions |
| Linear Algebra | vandermonde, toeplitz, hankel | fm_structured_matrix |
| Combinatorics | permutation, subset, partition | fm_combinatorics |
| ODE | rk4, euler, adams | fm_ode |

---

## Modernization Patterns

### COMMON Block → Module Variable

```fortran
! BEFORE (burkardt legacy)
COMMON /PARAMS/ TOL, MAXITER
REAL*8 TOL
INTEGER MAXITER

! AFTER (modern)
module fm_solver_params
  implicit none
  real(8), parameter :: default_tol = 1.0d-12
  integer, parameter :: default_maxiter = 1000
end module
```

### Implicit Typing → Explicit

```fortran
! BEFORE
      SUBROUTINE LEGCOMP(N,X,W)
      REAL*8 X(N),W(N)

! AFTER
subroutine legendre_compute(n, x, w)
  implicit none
  integer, intent(in) :: n
  real(8), intent(out) :: x(n), w(n)
```

### GOTO → Structured Control

```fortran
! BEFORE
      IF (X.LT.0) GOTO 100
      Y = SQRT(X)
      GOTO 200
  100 Y = 0.0D0
  200 CONTINUE

! AFTER
if (x < 0.0d0) then
  y = 0.0d0
else
  y = sqrt(x)
end if
```

---

## Workflows

### Integrating a New Burkardt Routine

1. Identify the routine and its category
2. Create or find existing `fm_{category}` module
3. Copy routine as internal (private) subroutine
4. Write public `bind(C)` wrapper with clean interface
5. Add to `fortran.zig` extern declarations
6. Write Zig test with known-answer validation
7. Run `zig build test`

---

## Anti-Patterns

- **Exporting burkardt names directly** — always wrap with fm_ prefix
- **Keeping COMMON blocks** — convert to module variables or parameters
- **Leaving implicit typing** — every file needs `implicit none`
- **One routine per module** — group by category (quadrature, interpolation, etc.)

---

## Cross-References

- [fortran-module-design](../fortran-module-design/SKILL.md) — Module organization
- [fortran-fixed-format](../fortran-fixed-format/SKILL.md) — Legacy F77 patterns
- [zortran-bind-c](../../zortran/zortran-bind-c/SKILL.md) — bind(C) export patterns
