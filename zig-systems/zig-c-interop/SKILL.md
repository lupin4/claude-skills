---
name: "zig-c-interop"
description: "Use when interfacing Zig with Fortran bind(C) or C libraries. Covers @cImport, translate-c, ABI-safe struct layouts, extern struct padding, and calling conventions."
---

# Zig C Interop

C ABI interop patterns for Zig — interfacing with Fortran bind(C) exports and third-party C libraries.

---

## @cImport vs Manual extern

**For Fortran bind(C): use manual extern fn** (no C header exists).

```zig
// Manual extern — preferred for Fortran
pub extern fn fm_compute(x: [*]const f64, n: c_int, result: *f64, status: *c_int) void;
```

**For C libraries with headers: use @cImport.**

```zig
const c = @cImport({
    @cInclude("fftw3.h");
});
// Now use c.fftw_plan, c.fftw_execute, etc.
```

## ABI-Safe Struct Layouts

```zig
// extern struct — matches C ABI layout exactly
const Vec3 = extern struct {
    x: f64,
    y: f64,
    z: f64,
};

// Verify size matches Fortran's bind(C) type
comptime {
    std.debug.assert(@sizeOf(Vec3) == 24); // 3 × 8 bytes
    std.debug.assert(@alignOf(Vec3) == 8);
}
```

**Rules:**
- Use `extern struct` for anything crossing the FFI boundary
- Never use `packed struct` for FFI (different ABI)
- Verify with `@sizeOf` and `@alignOf` at comptime

---

## Anti-Patterns

- Refer to domain-specific anti-patterns in the skill content above

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [fortran-kernels/](../../fortran-kernels/) — Fortran kernel conventions
