# Zig extern fn Patterns Reference

## ABI Safety Checklist

- [ ] All scalar `value` params map to bare Zig types (f64, c_int)
- [ ] All non-value scalar params map to pointers (*f64, *c_int)
- [ ] Arrays use `[*]` (many-item pointer), not `*` (single-item)
- [ ] Read-only arrays use `[*]const`
- [ ] Fixed-size arrays use `*const [N]f64` or `*[N]f64`
- [ ] extern structs have no padding gaps (verified with @sizeOf)
- [ ] Return types match: Fortran function → Zig return, subroutine → void
- [ ] Status output params are `*c_int`

## Type Correspondence Quick Reference

| Fortran bind(C) | extern fn Zig | Safe wrapper Zig |
|-----------------|---------------|------------------|
| `integer(c_int), value` | `c_int` | `i32` or `usize` |
| `integer(c_int), intent(out)` | `*c_int` | (mapped to error) |
| `real(c_double), value` | `f64` | `f64` |
| `real(c_double), intent(in) :: x(n)` | `[*]const f64` | `[]const f64` |
| `real(c_double), intent(inout) :: x(n)` | `[*]f64` | `[]f64` |
| `type(ft_vec3_t), intent(in)` | `*const Vec3` | `Vec3` (by value) |
| `type(c_ptr)` | `*anyopaque` | domain-specific pointer |
| `character(c_char) :: s(*)` | `[*]const u8` | `[]const u8` |

## Safe Wrapper Template

```zig
const fortran = @import("fortran.zig");
const std = @import("std");

pub const DomainError = error{
    InvalidInput,
    ComputationFailed,
    OutOfMemory,
};

fn mapStatus(status: c_int) DomainError!void {
    return switch (status) {
        0 => {},
        -1 => error.InvalidInput,
        -2 => error.ComputationFailed,
        -3 => error.OutOfMemory,
        else => error.InvalidInput,
    };
}

/// Brief description of what this does.
pub fn kernelName(input: []const f64, output: []f64) DomainError!void {
    std.debug.assert(input.len == output.len);
    var status: c_int = 0;
    fortran.fm_kernel_name(
        input.ptr,
        output.ptr,
        @intCast(input.len),
        &status,
    );
    try mapStatus(status);
}
```

## Column-Major ↔ Row-Major Index Conversion

```
Column-major (Fortran): index = row + col * num_rows
Row-major (C/Zig):      index = row * num_cols + col

Transpose relationship:
  fortran_mat(i, j) == zig_flat[i + j * rows]  (0-indexed)
  fortran_mat(i, j) == zig_flat[(i-1) + (j-1) * rows]  (1-indexed Fortran)
```
