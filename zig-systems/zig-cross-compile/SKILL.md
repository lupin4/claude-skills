---
name: "zig-cross-compile"
description: "Use when cross-compiling Fortran+Zig projects. Covers target triples, cpu features, sysroot setup, gfortran vs flang, and linker flags per platform."
---

# Zig Cross-Compile

Cross-compilation patterns for Fortran+Zig targeting aarch64-linux, x86_64-linux, x86_64-macos, and aarch64-macos.

---

## Target Triples

| Target | Triple | Notes |
|--------|--------|-------|
| Linux x86_64 | x86_64-linux-gnu | Most CI/cloud |
| Linux aarch64 | aarch64-linux-gnu | Jetson, Graviton |
| macOS x86_64 | x86_64-macos | Intel Macs |
| macOS aarch64 | aarch64-macos | Apple Silicon |

## build.zig Cross-Compile

```zig
const target = b.standardTargetOptions(.{});
// User selects: zig build -Dtarget=aarch64-linux-gnu

const lib = b.addStaticLibrary(.{
    .name = "forcore",
    .root_source_file = b.path("src/zig/lib.zig"),
    .target = target,
    .optimize = optimize,
});
```

## Fortran Cross-Compilation

Zig handles cross-compilation natively, but Fortran requires a cross-compiler:

```bash
# Native Fortran (always works)
gfortran -c -O2 -fPIC src/fortran/fm_core.f90

# Cross-compile Fortran for aarch64-linux
aarch64-linux-gnu-gfortran -c -O2 -fPIC src/fortran/fm_core.f90

# Or use flang (LLVM-based, better cross-compile support)
flang -c -O2 -fPIC --target=aarch64-linux-gnu src/fortran/fm_core.f90
```

---

## Anti-Patterns

- Refer to domain-specific anti-patterns in the skill content above

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [fortran-kernels/](../../fortran-kernels/) — Fortran kernel conventions
