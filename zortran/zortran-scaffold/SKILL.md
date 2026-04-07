---
name: "zortran-scaffold"
description: "Use when creating a new forKernels repo. Covers fm_/ft_/fml_ prefix conventions, build.zig skeleton, DEPENDENCY_RULES.md template, and standard directory layout."
---

# Zortran Scaffold

Scaffolding skill for new forKernels repositories. Generates the standard directory structure, build.zig configuration, naming conventions, and dependency governance files that every Zortran repo needs.

---

## Table of Contents

- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [Naming Conventions](#naming-conventions)
- [build.zig Skeleton](#buildzig-skeleton)
- [DEPENDENCY_RULES.md Template](#dependency_rulesmd-template)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

Provides the canonical repo skeleton for any new forKernels project — the directory layout, file naming prefixes, build system boilerplate, and dependency governance. Every repo in the forKernels family (forSim, forTorch, forDSP, forSec, etc.) starts from this template.

### Prerequisites

- Zig 0.13+ installed
- gfortran or flang available on PATH
- Familiarity with Fortran `bind(C)` and Zig `extern fn`

---

## Directory Structure

```
for{Name}/
├── build.zig              # Single build entry point
├── DEPENDENCY_RULES.md    # Archive tier governance
├── README.md
├── src/
│   ├── fortran/           # Fortran source (.f90)
│   │   ├── fm_{name}_core.f90
│   │   └── fm_{name}_utils.f90
│   ├── zig/               # Zig source
│   │   ├── main.zig       # Entry point (if executable)
│   │   ├── fortran.zig    # extern fn declarations for Fortran
│   │   └── lib.zig        # Public Zig API
│   └── c/                 # C headers (if needed for third-party)
├── tests/
│   ├── test_kernels.zig   # Zig test blocks calling Fortran
│   └── test_data/         # Reference data for validation
├── lib/                   # Pre-built .a archives (tier dependencies)
└── docs/                  # Design notes, API docs
```

## Naming Conventions

### Function Prefixes

| Prefix | Scope | Example |
|--------|-------|---------|
| `fm_` | Module-level function | `fm_rigid_body_step` |
| `ft_` | Type definition | `ft_tensor_t`, `ft_quaternion_t` |
| `fml_` | ML-specific function | `fml_forward_pass` |
| `fs_` | Subroutine (void return) | `fs_init_scene` |

### File Naming

```
fm_{domain}_{operation}.f90    # Fortran module file
ft_{domain}_types.f90          # Type definitions module
test_{domain}.zig              # Zig test file
```

### Module Naming

```fortran
module fm_rigid_body
  use iso_c_binding
  implicit none
  private
  ! Only bind(C) wrappers are public
  public :: fm_rigid_body_step, fm_rigid_body_init
contains
  ! ...
end module fm_rigid_body
```

---

## build.zig Skeleton

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // --- Fortran static library ---
    const fortran_lib = b.addSystemCommand(&.{
        "gfortran",
        "-c", "-O2", "-fPIC",
        "-J", "zig-out/mod",           // .mod output dir
        "src/fortran/fm_core.f90",
    });

    const ar_cmd = b.addSystemCommand(&.{
        "ar", "rcs",
        "zig-out/lib/libforcore.a",
        "zig-out/obj/fm_core.o",
    });
    ar_cmd.step.dependOn(&fortran_lib.step);

    // --- Zig library wrapping Fortran ---
    const lib = b.addStaticLibrary(.{
        .name = "forcore",
        .root_source_file = b.path("src/zig/lib.zig"),
        .target = target,
        .optimize = optimize,
    });
    lib.addLibraryPath(b.path("zig-out/lib"));
    lib.linkSystemLibrary("forcore");
    lib.linkSystemLibrary("gfortran");
    lib.step.dependOn(&ar_cmd.step);

    b.installArtifact(lib);

    // --- Tests ---
    const tests = b.addTest(.{
        .root_source_file = b.path("tests/test_kernels.zig"),
        .target = target,
        .optimize = optimize,
    });
    tests.addLibraryPath(b.path("zig-out/lib"));
    tests.linkSystemLibrary("forcore");
    tests.linkSystemLibrary("gfortran");
    tests.step.dependOn(&ar_cmd.step);

    const run_tests = b.addRunArtifact(tests);
    const test_step = b.step("test", "Run kernel tests");
    test_step.dependOn(&run_tests.step);
}
```

---

## DEPENDENCY_RULES.md Template

```markdown
# Dependency Rules

## Archive Tiers

| Tier | Archives | Update Policy |
|------|----------|---------------|
| 1 (core) | libformath.a, libforutil.a | Stable API, semver |
| 2 (domain) | libforsim.a, libfordsp.a | Domain-specific, may evolve |
| 3 (experimental) | libforexp_*.a | No stability guarantee |

## Rules

1. Tier-1 archives may NOT depend on tier-2 or tier-3
2. Tier-2 archives may depend on tier-1 only
3. Tier-3 archives may depend on tier-1 and tier-2
4. Circular dependencies are forbidden
5. Link order follows reverse topological sort of dependency graph
6. New archives require an entry in this file before merge

## Current Dependency Graph

```
libformath.a (tier-1) ← no deps
libforutil.a (tier-1) ← libformath.a
libforsim.a  (tier-2) ← libformath.a, libforutil.a
```
```

---

## Workflows

### New Repo from Scratch

1. Copy directory structure template
2. Set project name in build.zig
3. Create initial Fortran module with `bind(C)` exports
4. Create matching `fortran.zig` with `extern fn` declarations
5. Write first Zig test calling the Fortran kernel
6. Run `zig build test` to validate the full pipeline
7. Fill in DEPENDENCY_RULES.md with archive tier assignments

### Adding a New Kernel

1. Create `src/fortran/fm_{domain}_{operation}.f90`
2. Add `bind(C)` wrappers for all public routines
3. Add `extern fn` declarations in `src/zig/fortran.zig`
4. Add compile step in `build.zig`
5. Write test in `tests/test_{domain}.zig`
6. Run symbol sync check (see zortran-symbol-sync)

---

## Anti-Patterns

- **CMake or Makefiles** — build.zig is the only build system
- **Mixing prefixes** — `fm_` is for functions, `ft_` for types, never interchange
- **Skipping DEPENDENCY_RULES.md** — every archive must be documented before merge
- **Direct C headers for Fortran** — use `bind(C)` + Zig `extern fn`, not manual C wrappers
- **Non-snake_case naming** — all Fortran identifiers are snake_case

---

## Cross-References

- [zortran-bind-c](../zortran-bind-c/SKILL.md) — Fortran `bind(C)` export patterns
- [zortran-zig-wrapper](../zortran-zig-wrapper/SKILL.md) — Zig wrapper conventions
- [zortran-archive-tiers](../zortran-archive-tiers/SKILL.md) — Archive tier strategy
- [zig-build-system](../../zig-systems/zig-build-system/SKILL.md) — Advanced build.zig patterns
