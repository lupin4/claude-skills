# Zortran Core Patterns — Claude Code Guidance

Fortran+Zig interop skills for the forKernels ecosystem. "Zortran" = the pattern of writing numerical kernels in Fortran with `bind(C)` exports, wrapped by safe Zig interfaces via `extern fn`.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **zortran-scaffold** | New repo skeleton: fm_/ft_/fml_ prefixes, build.zig, DEPENDENCY_RULES.md |
| **zortran-bind-c** | Fortran `bind(C)` exports, iso_c_binding idioms, type mapping |
| **zortran-zig-wrapper** | Zig `extern fn` declarations, safe wrappers, error unions |
| **zortran-archive-tiers** | Tier-1/2/3 archive strategy, BSD vs GNU ar, link order |
| **zortran-test-harness** | Zig test blocks for Fortran kernel validation |
| **zortran-symbol-sync** | Keeping fortran.zig extern fn count in sync with bind(C) exports |

## Key Conventions

- **Prefixes:** `fm_` (module/function), `ft_` (type), `fml_` (ML-specific)
- **Build:** All repos use `build.zig` — no CMake, no Makefiles
- **Archives:** Static `.a` libraries organized in tier-1 (core), tier-2 (domain), tier-3 (experimental)
- **Interop contract:** Fortran `bind(C)` → C ABI → Zig `extern fn`
- **Testing:** Zig `test` blocks call Fortran kernels directly

## Cross-References

- fortran-kernels/ — Numerical kernel patterns (LAPACK, Lie groups, SIMD)
- zig-systems/ — Zig build system, allocators, cross-compilation
- cybersecurity/ — forSec build architecture (18 sibling archives)
