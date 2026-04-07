---
name: "zig-comptime-dispatch"
description: "Use when implementing comptime type dispatch for kernel family switching in Zig. Covers comptime if/switch, generic interfaces, inline for, and comptime string manipulation."
---

# Zig Comptime Dispatch

Comptime type dispatch patterns for selecting kernel implementations at compile time based on precision, algorithm family, or target architecture.

---

## Basic Comptime Dispatch

```zig
fn kernel(comptime T: type, x: []T, y: []T) void {
    if (T == f32) {
        // Call single-precision Fortran kernel
        fortran.fm_kernel_f32(x.ptr, y.ptr, @intCast(x.len));
    } else if (T == f64) {
        // Call double-precision Fortran kernel
        fortran.fm_kernel_f64(x.ptr, y.ptr, @intCast(x.len));
    } else {
        @compileError("Unsupported type for kernel: " ++ @typeName(T));
    }
}
```

## Kernel Family Switching

```zig
const KernelFamily = enum { blas, custom, fallback };

fn matmul(comptime family: KernelFamily, a: []const f64, b: []const f64, c: []f64, m: usize, n: usize, k: usize) void {
    switch (family) {
        .blas => fortran.dgemm('N', 'N', @intCast(m), @intCast(n), @intCast(k), 1.0, a.ptr, @intCast(m), b.ptr, @intCast(k), 0.0, c.ptr, @intCast(m)),
        .custom => customMatmul(a, b, c, m, n, k),
        .fallback => naiveMatmul(a, b, c, m, n, k),
    }
}
```

## Inline For Over Type Lists

```zig
const supported_types = .{ f32, f64, f128 };

inline for (supported_types) |T| {
    if (T == requested_type) {
        return kernel(T, data);
    }
}
```

---

## Anti-Patterns

- Refer to domain-specific anti-patterns in the skill content above

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [fortran-kernels/](../../fortran-kernels/) — Fortran kernel conventions
