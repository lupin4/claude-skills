---
name: "zig-build-system"
description: "Use when configuring build.zig for Fortran+Zig projects. Covers step chaining, addStaticLibrary, system commands for ar/ranlib, custom build steps, and conditional compilation."
---

# Zig Build System

Build.zig patterns for the forKernels ecosystem — compiling Fortran, creating archives, linking, and running tests through Zig's build system.

---

## Step Dependency Chain

```
Fortran compile → ar archive → Zig library (links archive) → Tests (links library)
```

### Complete build.zig

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // Step 1: Compile Fortran sources
    const fortran_compile = b.addSystemCommand(&.{
        "gfortran", "-c", "-O2", "-fPIC",
        "-J", "zig-out/mod",
        "src/fortran/fm_core.f90",
        "-o", "zig-out/obj/fm_core.o",
    });

    // Step 2: Create archive
    const ar = b.addSystemCommand(&.{
        "ar", "rcs",
        "zig-out/lib/libforcore.a",
        "zig-out/obj/fm_core.o",
    });
    ar.step.dependOn(&fortran_compile.step);

    // Step 3: Zig library
    const lib = b.addStaticLibrary(.{
        .name = "forcore_zig",
        .root_source_file = b.path("src/zig/lib.zig"),
        .target = target,
        .optimize = optimize,
    });
    lib.addLibraryPath(b.path("zig-out/lib"));
    lib.linkSystemLibrary("forcore");
    lib.linkSystemLibrary("gfortran");
    lib.step.dependOn(&ar.step);
    b.installArtifact(lib);

    // Step 4: Tests
    const tests = b.addTest(.{
        .root_source_file = b.path("tests/test_kernels.zig"),
        .target = target,
        .optimize = optimize,
    });
    tests.addLibraryPath(b.path("zig-out/lib"));
    tests.linkSystemLibrary("forcore");
    tests.linkSystemLibrary("gfortran");
    tests.step.dependOn(&ar.step);

    const test_step = b.step("test", "Run kernel validation tests");
    test_step.dependOn(&b.addRunArtifact(tests).step);
}
```

---

## Anti-Patterns

- Refer to domain-specific anti-patterns in the skill content above

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [fortran-kernels/](../../fortran-kernels/) — Fortran kernel conventions
