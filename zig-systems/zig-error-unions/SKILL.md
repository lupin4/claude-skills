---
name: "zig-error-unions"
description: "Use when mapping Fortran INTEGER status codes to Zig error sets. Covers error union return patterns, errdefer cleanup, custom error sets, and try/catch patterns."
---

# Zig Error Unions

Error set design for wrapping Fortran return codes in idiomatic Zig error unions.

---

## Status Code Convention

Fortran kernels return status via `integer(c_int), intent(out) :: status`:
- **0** = success
- **Negative** = error (specific code per kernel)
- **Positive** = warning/info

## Error Set Pattern

```zig
pub const LinAlgError = error{
    InvalidInput,       // -1
    SingularMatrix,     // -2
    NotConverged,       // -3
    OutOfMemory,        // -4
    DimensionMismatch,  // -5
};

fn mapLinAlgStatus(status: c_int) LinAlgError!void {
    return switch (status) {
        0 => {},
        -1 => error.InvalidInput,
        -2 => error.SingularMatrix,
        -3 => error.NotConverged,
        -4 => error.OutOfMemory,
        -5 => error.DimensionMismatch,
        else => error.InvalidInput,
    };
}
```

## errdefer for Cleanup

```zig
pub fn createSolver(allocator: std.mem.Allocator, n: usize) !*Solver {
    const workspace = try allocator.alloc(f64, n * n);
    errdefer allocator.free(workspace); // Free on error only

    var status: c_int = 0;
    fortran.fm_solver_init(workspace.ptr, @intCast(n), &status);
    try mapLinAlgStatus(status);

    return &Solver{ .workspace = workspace, .n = n };
}
```

---

## Anti-Patterns

- Refer to domain-specific anti-patterns in the skill content above

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [fortran-kernels/](../../fortran-kernels/) — Fortran kernel conventions
