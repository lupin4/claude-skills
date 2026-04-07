---
name: "fortran-fixed-format"
description: "Use when maintaining legacy F77 fixed-format code. Covers column rules, continuation lines, COMMON blocks, EQUIVALENCE elimination, and modernization paths to F2008."
---

# Fortran Fixed Format

Legacy F77 maintenance and modernization. Covers the column-based formatting rules, continuation lines, COMMON block patterns, and systematic migration paths to free-format F2008+.

---

## Overview

### Column Rules (Fixed Format)

| Columns | Purpose |
|---------|---------|
| 1 | Comment (`C`, `c`, `*`, `!`) |
| 1-5 | Statement label (numeric) |
| 6 | Continuation character (any non-blank, non-zero) |
| 7-72 | Statement body |
| 73-80 | Sequence number (ignored by compiler) |

### Continuation Lines

```fortran
C     This is a comment (column 1)
      RESULT = A + B + C
     &       + D + E + F
     &       + G + H
```

The `&` (or any non-blank character) in column 6 marks a continuation.

---

## Migration Path: F77 → F2008

### Phase 1: Mechanical Conversion

1. Convert `.f` → `.f90` (free format)
2. Remove column restrictions
3. Convert `C` comments → `!` comments
4. Add `implicit none` to every program unit
5. Add `intent` to all arguments

### Phase 2: Structural Modernization

1. Replace `COMMON` blocks with modules
2. Replace `EQUIVALENCE` with explicit variables
3. Replace `GOTO` with structured control flow
4. Replace `REAL*8` with `real(8)` or `real(c_double)`
5. Replace `CHARACTER*N` with `character(len=N)`

### Phase 3: Interface Modernization

1. Add `bind(C)` wrappers for public routines
2. Create module interfaces
3. Add allocatable arrays (replace static sizing)

---

## COMMON Block to Module Migration

```fortran
! BEFORE (F77)
      COMMON /SIMPARAMS/ DT, GRAV, MAXSTEP
      REAL*8 DT, GRAV
      INTEGER MAXSTEP

! AFTER (F2008)
module fm_sim_params
  use iso_c_binding
  implicit none
  real(c_double) :: dt = 0.001d0
  real(c_double) :: grav = 9.81d0
  integer(c_int) :: maxstep = 10000
end module
```

---

## Anti-Patterns

- **Leaving implicit typing** — add `implicit none` everywhere, no exceptions
- **Partial migration** — convert whole files, not individual routines
- **Keeping EQUIVALENCE** — always replace with explicit variables
- **Preserving sequence numbers** — columns 73-80 are dead weight in free format

---

## Cross-References

- [fortran-burkhardt](../fortran-burkhardt/SKILL.md) — Many burkardt routines need modernization
- [fortran-module-design](../fortran-module-design/SKILL.md) — Target module patterns
