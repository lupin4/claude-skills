---
name: "zortran-archive-tiers"
description: "Use when managing static library archives in the forKernels ecosystem. Covers tier-1/2/3 classification, BSD vs GNU ar macOS fix, link order, and DEPENDENCY_RULES.md governance."
---

# Zortran Archive Tiers

Static library archive management for the forKernels ecosystem. Archives (`.a` files) are classified into three tiers with strict dependency rules enforced by DEPENDENCY_RULES.md. Includes the BSD ar vs GNU ar macOS compatibility fix.

---

## Table of Contents

- [Overview](#overview)
- [Tier Definitions](#tier-definitions)
- [Link Order](#link-order)
- [BSD ar vs GNU ar (macOS Fix)](#bsd-ar-vs-gnu-ar-macos-fix)
- [build.zig Integration](#buildzig-integration)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

Manages the classification, dependency rules, and linking of `.a` static archives across the forKernels ecosystem. The tier system prevents circular dependencies and ensures stable APIs at the core level while allowing experimentation at the edges.

---

## Tier Definitions

| Tier | Purpose | Stability | May Depend On |
|------|---------|-----------|---------------|
| **Tier-1 (core)** | Math primitives, utilities, types | Stable API, semver | Nothing (leaf nodes) |
| **Tier-2 (domain)** | Domain-specific kernels (sim, DSP, ML) | Domain-stable, may evolve | Tier-1 only |
| **Tier-3 (experimental)** | Research, prototypes, unstable | No guarantees | Tier-1 and Tier-2 |

### Examples

```
Tier-1: libformath.a, libforutil.a, libfortypes.a
Tier-2: libforsim.a, libfordsp.a, libforml.a, libforsec.a
Tier-3: libforexp_newquant.a, libforexp_hopfield.a
```

### Tier Decision Tree

```
Is it a math/utility primitive used by 3+ repos?
  → YES → Tier-1
  → NO  → Is it a domain-specific kernel family?
           → YES → Tier-2
           → NO  → Tier-3 (experimental)
```

---

## Link Order

Archives must be linked in **reverse topological order** — dependents before dependencies:

```bash
# WRONG: dependency before dependent
-lformath -lforsim    # forsim can't find formath symbols

# RIGHT: dependent first, then its dependencies
-lforsim -lformath    # linker resolves forsim's references in formath
```

### Full Link Order Example (18 archives)

```
# Tier-3 first (most dependent)
-lforexp_hopfield -lforexp_newquant
# Tier-2 next
-lforsec -lfordsp -lforml -lforsim -lfortorch
# Tier-1 last (least dependent)
-lforutil -lformath -lfortypes
# System libraries
-lgfortran -lm
```

### Topological Sort in build.zig

```zig
const archives = [_][]const u8{
    // Tier-3
    "forexp_hopfield", "forexp_newquant",
    // Tier-2
    "forsec", "fordsp", "forml", "forsim", "fortorch",
    // Tier-1
    "forutil", "formath", "fortypes",
};

for (archives) |name| {
    lib.linkSystemLibrary(name);
}
lib.linkSystemLibrary("gfortran");
```

---

## BSD ar vs GNU ar (macOS Fix)

### The Problem

macOS ships BSD `ar` which creates thin archives by default. GNU `ar` (from binutils) creates fat archives. When cross-compiling or using Homebrew gfortran on macOS, archive format mismatches cause:

```
error: archive member 'fm_core.o' is not a valid object file
```

### The Fix

```bash
# Check which ar you have
ar --version
# BSD: no --version flag, prints usage
# GNU: "GNU ar (GNU Binutils) 2.x"

# macOS: force creation of non-thin archives
ar -rcs libforcore.a fm_core.o    # BSD ar: works (always fat)
ar rcsD libforcore.a fm_core.o    # GNU ar with deterministic mode

# If using Homebrew GNU ar alongside BSD ar:
/usr/bin/ar -rcs libforcore.a fm_core.o         # BSD (system)
/opt/homebrew/bin/gar -rcs libforcore.a fm_core.o  # GNU (Homebrew)
```

### build.zig Fix

```zig
const ar_cmd = if (builtin.os.tag == .macos)
    b.addSystemCommand(&.{ "/usr/bin/ar", "rcs" })  // Force BSD ar
else
    b.addSystemCommand(&.{ "ar", "rcs" });

ar_cmd.addArg(b.fmt("zig-out/lib/lib{s}.a", .{name}));
ar_cmd.addArg(b.fmt("zig-out/obj/{s}.o", .{name}));
```

---

## build.zig Integration

### Multi-Archive Linking

```zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const lib = b.addStaticLibrary(.{
        .name = "myproject",
        .root_source_file = b.path("src/zig/lib.zig"),
        .target = target,
        .optimize = optimize,
    });

    // Add archive search paths
    lib.addLibraryPath(b.path("lib/tier1"));
    lib.addLibraryPath(b.path("lib/tier2"));

    // Link in reverse topological order
    lib.linkSystemLibrary("forsim");    // tier-2
    lib.linkSystemLibrary("formath");   // tier-1
    lib.linkSystemLibrary("gfortran");  // system

    b.installArtifact(lib);
}
```

---

## Workflows

### Adding a New Archive

1. Determine tier (1, 2, or 3) using decision tree
2. Add entry to DEPENDENCY_RULES.md with dependencies listed
3. Verify no circular dependencies in the graph
4. Add to build.zig in correct link order position
5. Test linking on both Linux and macOS

### Promoting Tier-3 → Tier-2

1. Stabilize API — no breaking changes for 2+ releases
2. Verify it only depends on tier-1 archives
3. Update DEPENDENCY_RULES.md tier classification
4. Move from `lib/tier3/` to `lib/tier2/` if using directory-based organization
5. Update all downstream build.zig files

---

## Anti-Patterns

- **Circular dependencies** — tier-1 depending on tier-2 is always wrong
- **Skipping DEPENDENCY_RULES.md** — undocumented archives become tech debt
- **Wrong link order** — causes "undefined symbol" errors that look like missing code
- **Mixing BSD/GNU ar on macOS** — causes corrupt archive errors
- **Fat archives in git** — use git-lfs or build from source in CI

---

## Cross-References

- [zortran-scaffold](../zortran-scaffold/SKILL.md) — DEPENDENCY_RULES.md template
- [zortran-symbol-sync](../zortran-symbol-sync/SKILL.md) — Symbol inventory
- [forsec-build-arch](../../cybersecurity/forsec-build-arch/SKILL.md) — 18-archive build
- [zig-build-system](../../zig-systems/zig-build-system/SKILL.md) — build.zig patterns
