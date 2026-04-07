# iso_c_binding Complete Reference

## Type Mapping Table

| Fortran Kind | iso_c_binding Constant | C Type | Zig Type | Size (bytes) |
|-------------|----------------------|--------|----------|-------------|
| `integer(c_int)` | `c_int` | `int` | `c_int` | 4 |
| `integer(c_short)` | `c_short` | `short` | `c_short` | 2 |
| `integer(c_long)` | `c_long` | `long` | `c_long` | 4/8 |
| `integer(c_long_long)` | `c_long_long` | `long long` | `c_longlong` | 8 |
| `integer(c_int8_t)` | `c_int8_t` | `int8_t` | `i8` | 1 |
| `integer(c_int16_t)` | `c_int16_t` | `int16_t` | `i16` | 2 |
| `integer(c_int32_t)` | `c_int32_t` | `int32_t` | `i32` | 4 |
| `integer(c_int64_t)` | `c_int64_t` | `int64_t` | `i64` | 8 |
| `integer(c_size_t)` | `c_size_t` | `size_t` | `usize` | ptr-size |
| `real(c_float)` | `c_float` | `float` | `f32` | 4 |
| `real(c_double)` | `c_double` | `double` | `f64` | 8 |
| `real(c_long_double)` | `c_long_double` | `long double` | `c_longdouble` | 10/16 |
| `logical(c_bool)` | `c_bool` | `_Bool` | `bool` | 1 |
| `character(c_char)` | `c_char` | `char` | `u8` | 1 |
| `type(c_ptr)` | `c_ptr` | `void*` | `*anyopaque` | ptr-size |
| `type(c_funptr)` | `c_funptr` | `void(*)()` | `*const fn()` | ptr-size |

## Named Constants

| Constant | Value | Use |
|----------|-------|-----|
| `c_null_ptr` | null | Initialize `type(c_ptr)` to null |
| `c_null_funptr` | null | Initialize `type(c_funptr)` to null |
| `c_null_char` | `'\0'` | Null terminator for C strings |
| `c_new_line` | `'\n'` | Newline character |
| `c_alert` | `'\a'` | Alert/bell character |

## Utility Procedures

| Procedure | Signature | Purpose |
|-----------|-----------|---------|
| `c_loc(x)` | `type(c_ptr)` | Get C pointer to Fortran variable (must have TARGET) |
| `c_funloc(f)` | `type(c_funptr)` | Get C function pointer to Fortran procedure |
| `c_f_pointer(cptr, fptr [, shape])` | — | Convert C pointer to Fortran pointer |
| `c_f_procpointer(cfunptr, fptr)` | — | Convert C function pointer to Fortran procedure pointer |
| `c_associated(cptr [, cptr2])` | `logical` | Test if C pointer is associated (not null) |
| `c_sizeof(x)` | `integer(c_size_t)` | Size in bytes |

## Common Pitfalls

### 1. Missing `value` Attribute

```fortran
! WRONG: n is passed as pointer (int*)
subroutine foo(n) bind(C)
  integer(c_int), intent(in) :: n  ! This is int*, not int

! RIGHT: n is passed by value (int)
subroutine foo(n) bind(C)
  integer(c_int), intent(in), value :: n  ! This is int
```

### 2. Column-Major vs Row-Major

```
Fortran array(3,4):  Memory: [a(1,1), a(2,1), a(3,1), a(1,2), ...]
C/Zig array[4][3]:   Memory: [a[0][0], a[0][1], a[0][2], a[1][0], ...]

They are TRANSPOSED. Either:
- Transpose in the wrapper
- Document and handle in Zig
- Use 1D arrays with explicit indexing
```

### 3. String Length Mismatch

```fortran
! Fortran strings are space-padded, NOT null-terminated
character(len=10) :: s = "hello"  ! s = "hello     " (space-padded)

! For C interop, always null-terminate or pass length
```

### 4. Assumed-Shape Arrays Not Allowed

```fortran
! WRONG: assumed-shape not compatible with bind(C)
subroutine foo(x) bind(C)
  real(c_double), intent(in) :: x(:)

! RIGHT: explicit size
subroutine foo(x, n) bind(C)
  integer(c_int), intent(in), value :: n
  real(c_double), intent(in) :: x(n)
```

### 5. Optional Arguments Not Allowed

```fortran
! WRONG: optional not compatible with bind(C)
subroutine foo(x, tol) bind(C)
  real(c_double), intent(in), value :: x
  real(c_double), intent(in), value, optional :: tol  ! COMPILE ERROR

! RIGHT: use sentinel value
subroutine foo(x, tol) bind(C)
  real(c_double), intent(in), value :: x
  real(c_double), intent(in), value :: tol  ! Use -1.0 as "not specified"
```
