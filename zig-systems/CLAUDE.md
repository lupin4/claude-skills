# Zig Systems Layer — Claude Code Guidance

Zig infrastructure skills for the forKernels ecosystem — build system, memory management, cross-compilation, and C/Fortran interop.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **zig-build-system** | build.zig step chaining, addStaticLibrary, custom steps |
| **zig-comptime-dispatch** | Comptime type dispatch for kernel family switching |
| **zig-c-interop** | @cImport, translate-c, ABI-safe struct layouts |
| **zig-error-unions** | Error set design for wrapping Fortran return codes |
| **zig-allocator-patterns** | Arena/GPA/FixedBuffer strategies for kernel pipelines |
| **zig-cross-compile** | Target triples, cpu features, linker flags per platform |

## Key Conventions

- **Zig version:** 0.13+ (stable)
- **Build:** `build.zig` is the single build entry point — no shell scripts
- **Extern:** Manual `extern fn` declarations preferred over `@cImport` for Fortran
- **Errors:** Fortran INTEGER status codes → Zig error sets via explicit mapping
- **Memory:** ArenaAllocator for batch kernel runs, GPA for development/debug
- **Targets:** aarch64-linux-gnu, x86_64-linux-gnu, x86_64-macos, aarch64-macos

## Cross-References

- zortran/ — Fortran bind(C) ↔ Zig extern fn patterns
- cybersecurity/ — forSec build architecture (multi-archive linking)
- fortran-kernels/ — The kernels being wrapped
