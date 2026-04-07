# Fortran Numerical Kernels — Claude Code Guidance

Numerical computing skills for writing production Fortran kernels — from legacy F77 maintenance to modern F2008+ with SIMD vectorization and GPU offload.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **fortran-burkhardt** | Integrating burkardt-modern routines, module wrapping |
| **fortran-lapack-blas** | LAPACK/BLAS linking, interface blocks, vendor implementations |
| **fortran-lie-groups** | SO(3)/SE(3), quaternion, dual quaternion kernels for robotics |
| **fortran-fixed-format** | Legacy F77 maintenance, COMMON blocks, modernization paths |
| **fortran-simd-hints** | Vectorization directives, DO CONCURRENT, OpenMP offload |
| **fortran-module-design** | USE/ONLY, IMPLICIT NONE, public/private, submodules |

## Key Conventions

- **Standard:** F2008+ for new code, F77 tolerance for legacy maintenance
- **Module pattern:** One module per file, PRIVATE by default, explicit ONLY imports
- **Naming:** snake_case for routines, UPPER_CASE for constants, `_t` suffix for derived types
- **Interop:** All public kernels get `bind(C)` wrappers (see zortran/)
- **Testing:** Validated via Zig test harness (see zortran/zortran-test-harness)

## Cross-References

- zortran/ — Fortran↔Zig interop patterns
- ai-ml-kernels/ — ML primitives implemented in Fortran
- physics-sim/ — Physics simulation kernels
- signal-audio/ — DSP kernels (FFT, filters)
