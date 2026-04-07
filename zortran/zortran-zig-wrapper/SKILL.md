---
name: "zortran-zig-wrapper"
description: "Use when writing Zig wrappers for Fortran bind(C) exports. Covers extern fn declarations, safe wrappers with error unions, array-to-slice conversion, and ABI safety."
---

# Zortran Zig Wrapper

Safe Zig wrapper conventions for Fortran `bind(C)` exports. Every `extern fn` gets a safe Zig wrapper that converts raw pointers to slices, maps integer status codes to error unions, and provides idiomatic Zig interfaces.

---

## Table of Contents

- [Overview](#overview)
- [extern fn Declarations](#extern-fn-declarations)
- [Safe Wrapper Pattern](#safe-wrapper-pattern)
- [Error Union Design](#error-union-design)
- [Array to Slice Conversion](#array-to-slice-conversion)
- [Struct Mapping](#struct-mapping)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

The Zig wrapper layer sits between raw `extern fn` declarations and user-facing Zig code. Raw extern functions use C ABI types (`c_int`, `[*]f64`). Safe wrappers convert these to idiomatic Zig (`i32`, `[]f64`, error unions).

### The Two-Layer Pattern

```
User Zig code  →  Safe wrapper (lib.zig)  →  extern fn (fortran.zig)  →  Fortran bind(C)
                  - error unions              - raw C ABI types            - iso_c_binding
                  - slices                    - [*] pointers
                  - Zig idioms                - c_int, c_double
```

---

## extern fn Declarations

All Fortran exports are declared in a single `fortran.zig` file:

```zig
// src/zig/fortran.zig
// Keep in sync with Fortran bind(C) exports — see zortran-symbol-sync

const c = @cImport({}); // empty, just for c_int etc.

// fm_rigid_body_step(dt, positions, velocities, n, status)
pub extern fn fm_rigid_body_step(
    dt: f64,
    positions: [*]f64,
    velocities: [*]f64,
    n: c_int,
    status: *c_int,
) void;

// fm_quat_multiply(a, b, result)
pub extern fn fm_quat_multiply(
    a: *const [4]f64,
    b: *const [4]f64,
    result: *[4]f64,
) void;

// fm_fft_forward(data_re, data_im, n, status)
pub extern fn fm_fft_forward(
    data_re: [*]f64,
    data_im: [*]f64,
    n: c_int,
    status: *c_int,
) void;
```

**Rules:**
- One `fortran.zig` per repo, all externs in one place
- Comments show the Fortran signature for reference
- Use `[*]` for arrays (not `*`), `*const` for read-only
- Match Fortran `value` scalars to bare types, non-value to pointers

---

## Safe Wrapper Pattern

```zig
// src/zig/lib.zig
const fortran = @import("fortran.zig");

pub const RigidBodyError = error{
    InvalidSize,
    SingularMatrix,
    ConvergenceFailed,
};

/// Step rigid body simulation forward by dt seconds.
pub fn rigidBodyStep(
    dt: f64,
    positions: []f64,
    velocities: []f64,
) RigidBodyError!void {
    if (positions.len != velocities.len) return error.InvalidSize;
    if (positions.len == 0) return;

    var status: c_int = 0;
    const n: c_int = @intCast(positions.len);

    fortran.fm_rigid_body_step(
        dt,
        positions.ptr,
        velocities.ptr,
        n,
        &status,
    );

    return switch (status) {
        0 => {},
        -1 => error.SingularMatrix,
        -2 => error.ConvergenceFailed,
        else => error.InvalidSize,
    };
}
```

**Wrapper responsibilities:**
1. Convert slices to `[*]` pointers + length
2. Map Fortran status codes to Zig error unions
3. Validate inputs before calling Fortran
4. Use Zig naming (camelCase functions, PascalCase types)

---

## Error Union Design

### Status Code Mapping Convention

```zig
pub const KernelError = error{
    InvalidInput,      // status = -1
    SingularMatrix,    // status = -2
    ConvergenceFailed, // status = -3
    OutOfMemory,       // status = -4
    InvalidSize,       // status = -5
};

fn mapStatus(status: c_int) KernelError!void {
    return switch (status) {
        0 => {},
        -1 => error.InvalidInput,
        -2 => error.SingularMatrix,
        -3 => error.ConvergenceFailed,
        -4 => error.OutOfMemory,
        -5 => error.InvalidSize,
        else => error.InvalidInput, // unknown → generic
    };
}
```

**Convention:** Negative status = error, zero = success, positive = info/warning.

---

## Array to Slice Conversion

### Fortran 1D Array → Zig Slice

```zig
// Fortran: real(c_double), intent(inout) :: x(n)
// extern:  x: [*]f64, n: c_int
// Wrapper: x: []f64

pub fn compute(x: []f64) !f64 {
    var result: f64 = 0;
    var status: c_int = 0;
    fortran.fm_compute(x.ptr, @intCast(x.len), &result, &status);
    try mapStatus(status);
    return result;
}
```

### Fortran 2D Array → Zig Slice (Column-Major)

```zig
// Fortran stores column-major: mat(rows, cols)
// In memory: [col1_row1, col1_row2, ..., col2_row1, ...]
// Zig sees this as a flat slice of length rows*cols

pub fn matMul(
    a: []const f64, // rows_a × cols_a, column-major
    b: []const f64, // rows_b × cols_b, column-major
    c: []f64,       // rows_a × cols_b, column-major
    rows_a: usize,
    cols_a: usize,
    cols_b: usize,
) !void {
    // Validate dimensions
    if (a.len != rows_a * cols_a) return error.InvalidSize;
    if (b.len != cols_a * cols_b) return error.InvalidSize;
    if (c.len != rows_a * cols_b) return error.InvalidSize;

    var status: c_int = 0;
    fortran.fm_matmul(
        a.ptr, b.ptr, c.ptr,
        @intCast(rows_a), @intCast(cols_a), @intCast(cols_b),
        &status,
    );
    try mapStatus(status);
}
```

---

## Struct Mapping

```zig
// Fortran: type, bind(C) :: ft_vec3_t
//            real(c_double) :: x, y, z
//          end type

pub const Vec3 = extern struct {
    x: f64,
    y: f64,
    z: f64,

    // Safe Zig methods on top of Fortran struct
    pub fn normalize(self: *Vec3) void {
        fortran.fm_vec3_normalize(self);
    }

    pub fn dot(a: Vec3, b: Vec3) f64 {
        return a.x * b.x + a.y * b.y + a.z * b.z;
    }
};
```

---

## Workflows

### Adding a New Wrapper

1. Add `extern fn` to `fortran.zig` matching the Fortran `bind(C)` signature
2. Create safe wrapper in `lib.zig` with error union return
3. Convert `[*]` to slices, `c_int` to `usize` where appropriate
4. Map status codes to a domain-specific error set
5. Write Zig test calling the safe wrapper (not the raw extern)
6. Run `zig build test`

---

## Anti-Patterns

- **Exposing extern fn directly** — always wrap with safe Zig interface
- **Using @cImport for Fortran** — manual extern fn is clearer and more maintainable
- **Ignoring status codes** — always check and convert to error union
- **Casting without bounds check** — `@intCast` can panic; validate first
- **Mixing column-major/row-major** — document clearly, convert if needed

---

## Cross-References

- [zortran-bind-c](../zortran-bind-c/SKILL.md) — Fortran side of the interop
- [zortran-symbol-sync](../zortran-symbol-sync/SKILL.md) — Keeping in sync
- [zig-error-unions](../../zig-systems/zig-error-unions/SKILL.md) — Error set design patterns
