---
name: "zortran-bind-c"
description: "Use when writing Fortran bind(C) exports for Zig consumption. Covers iso_c_binding types, name mangling, string/array passing, derived type interop, and common pitfalls."
---

# Zortran bind(C)

Fortran `bind(C)` export patterns for the Zortran interop layer. Every public Fortran kernel must have a `bind(C)` wrapper to be callable from Zig. This skill covers the idioms, type mappings, and pitfalls of the iso_c_binding module.

---

## Table of Contents

- [Overview](#overview)
- [iso_c_binding Type Mapping](#iso_c_binding-type-mapping)
- [Function Signatures](#function-signatures)
- [String Passing](#string-passing)
- [Array Passing](#array-passing)
- [Derived Type Interop](#derived-type-interop)
- [Name Mangling](#name-mangling)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

Defines the conventions for exporting Fortran routines via `bind(C)` so they have a stable C ABI callable from Zig's `extern fn`. Covers scalar types, arrays, strings, derived types, and the name mangling rules that determine the symbol name in the object file.

### Prerequisites

- `use iso_c_binding` in all interop modules
- `implicit none` everywhere (non-negotiable)
- Understanding of C ABI calling conventions

---

## iso_c_binding Type Mapping

| Fortran | iso_c_binding | C equivalent | Zig equivalent |
|---------|--------------|--------------|----------------|
| `integer` | `integer(c_int)` | `int` | `c_int` |
| `integer(8)` | `integer(c_long)` | `long` | `c_long` |
| `integer(8)` | `integer(c_int64_t)` | `int64_t` | `i64` |
| `real` | `real(c_float)` | `float` | `f32` |
| `real(8)` | `real(c_double)` | `double` | `f64` |
| `logical` | `logical(c_bool)` | `_Bool` | `bool` |
| pointer | `type(c_ptr)` | `void*` | `*anyopaque` |
| func pointer | `type(c_funptr)` | `void(*)(...)` | `*const fn(...)` |
| `character` | `character(c_char)` | `char` | `u8` |

---

## Function Signatures

### Scalar Value Parameters

```fortran
function fm_add(a, b) result(c) bind(C, name="fm_add")
  use iso_c_binding
  real(c_double), intent(in), value :: a, b
  real(c_double) :: c
  c = a + b
end function
```

**Critical:** Use `value` attribute for scalar inputs. Without it, Fortran passes by reference (pointer), which changes the Zig signature from `f64` to `*const f64`.

### Output Parameters

```fortran
subroutine fm_compute(x, n, result, status) bind(C, name="fm_compute")
  use iso_c_binding
  integer(c_int), intent(in), value :: n
  real(c_double), intent(in) :: x(n)
  real(c_double), intent(out) :: result
  integer(c_int), intent(out) :: status
  result = sum(x)
  status = 0
end subroutine
```

### Void Returns (Subroutines)

```fortran
subroutine fs_init(ctx, n) bind(C, name="fs_init")
  use iso_c_binding
  type(c_ptr), intent(out) :: ctx
  integer(c_int), intent(in), value :: n
  ! ...
end subroutine
```

---

## String Passing

Fortran strings are NOT null-terminated. For C/Zig interop, always pass length explicitly or null-terminate manually.

### Pattern: Length-Prefixed String

```fortran
subroutine fm_log_message(msg, len) bind(C, name="fm_log_message")
  use iso_c_binding
  integer(c_int), intent(in), value :: len
  character(c_char), intent(in) :: msg(len)
  character(len=len) :: fstr
  integer :: i
  do i = 1, len
    fstr(i:i) = msg(i)
  end do
  ! Use fstr as normal Fortran string
end subroutine
```

### Pattern: Null-Terminated String

```fortran
subroutine fm_open_file(path) bind(C, name="fm_open_file")
  use iso_c_binding
  character(c_char), intent(in) :: path(*)
  character(len=256) :: fpath
  integer :: i
  fpath = ' '
  do i = 1, 256
    if (path(i) == c_null_char) exit
    fpath(i:i) = path(i)
  end do
  ! Use fpath(1:i-1)
end subroutine
```

---

## Array Passing

### Fixed-Size Arrays

```fortran
subroutine fm_mat3_det(mat, det) bind(C, name="fm_mat3_det")
  use iso_c_binding
  real(c_double), intent(in) :: mat(3,3)
  real(c_double), intent(out) :: det
  det = mat(1,1)*(mat(2,2)*mat(3,3) - mat(2,3)*mat(3,2)) &
      - mat(1,2)*(mat(2,1)*mat(3,3) - mat(2,3)*mat(3,1)) &
      + mat(1,3)*(mat(2,1)*mat(3,2) - mat(2,2)*mat(3,1))
end subroutine
```

### Variable-Size Arrays (pass length)

```fortran
subroutine fm_dot(x, y, n, result) bind(C, name="fm_dot")
  use iso_c_binding
  integer(c_int), intent(in), value :: n
  real(c_double), intent(in) :: x(n), y(n)
  real(c_double), intent(out) :: result
  result = dot_product(x(1:n), y(1:n))
end subroutine
```

**Memory layout:** Fortran is column-major. A `(3,4)` array in Fortran has the same memory layout as a `[4][3]` array in C/Zig. Account for this in Zig wrappers.

---

## Derived Type Interop

```fortran
type, bind(C) :: ft_vec3_t
  real(c_double) :: x, y, z
end type

subroutine fm_vec3_normalize(v) bind(C, name="fm_vec3_normalize")
  use iso_c_binding
  type(ft_vec3_t), intent(inout) :: v
  real(c_double) :: mag
  mag = sqrt(v%x**2 + v%y**2 + v%z**2)
  if (mag > 0.0d0) then
    v%x = v%x / mag
    v%y = v%y / mag
    v%z = v%z / mag
  end if
end subroutine
```

**Zig side:**
```zig
const Vec3 = extern struct {
    x: f64,
    y: f64,
    z: f64,
};

extern fn fm_vec3_normalize(v: *Vec3) void;
```

**Rules for `bind(C)` types:**
- No allocatable components
- No pointer components
- No type-bound procedures
- Only `iso_c_binding` kinds
- Struct layout matches C exactly (no padding surprises)

---

## Name Mangling

The `name=` clause in `bind(C)` controls the exact symbol name in the object file:

```fortran
! Symbol: fm_add (exact match)
function fm_add(a, b) bind(C, name="fm_add")

! Without name=, compiler may lowercase or add underscores
! NEVER rely on default mangling — always specify name=
function fm_add(a, b) bind(C)  ! BAD: symbol varies by compiler
```

**Verify with nm:**
```bash
nm -g libforcore.a | grep " T "
# Should show: fm_add, fm_compute, fs_init, etc.
```

---

## Workflows

### Adding a New bind(C) Export

1. Write the pure Fortran implementation (internal, not public)
2. Write the `bind(C)` wrapper subroutine/function calling the internal impl
3. Add `name="fm_{domain}_{operation}"` — always explicit
4. Use `value` for scalar inputs, bare arrays for array inputs
5. Return status via `integer(c_int), intent(out) :: status` parameter
6. Add matching `extern fn` in `fortran.zig` (see zortran-zig-wrapper)
7. Run symbol sync check

---

## Anti-Patterns

- **Omitting `name=`** — symbol names become compiler-dependent
- **Forgetting `value`** — scalars become pointers silently
- **Allocatable in bind(C) types** — not permitted, will fail to compile or produce ABI mismatch
- **Assumed-shape arrays `(:)`** — not compatible with C ABI, use explicit size `(n)`
- **Optional arguments** — not supported in bind(C), use sentinel values instead
- **Character(len=*)** — not C-compatible, use `character(c_char)` arrays

---

## Cross-References

- [zortran-zig-wrapper](../zortran-zig-wrapper/SKILL.md) — Zig side of the interop
- [zortran-symbol-sync](../zortran-symbol-sync/SKILL.md) — Keeping exports in sync
- [fortran-module-design](../../fortran-kernels/fortran-module-design/SKILL.md) — Module organization
