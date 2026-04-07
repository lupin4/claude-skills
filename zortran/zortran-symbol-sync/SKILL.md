---
name: "zortran-symbol-sync"
description: "Use when checking that Zig extern fn declarations match Fortran bind(C) exports. Covers symbol inventory with nm/objdump, drift detection, automation scripts, and CI integration."
---

# Zortran Symbol Sync

Keeping `fortran.zig` extern fn declarations in sync with Fortran `bind(C)` exports. Symbol drift — where Fortran adds/removes/renames an export without updating the Zig declarations — causes linker errors or silent ABI mismatches.

---

## Table of Contents

- [Overview](#overview)
- [Symbol Inventory](#symbol-inventory)
- [Drift Detection](#drift-detection)
- [Automation](#automation)
- [CI Integration](#ci-integration)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

Provides the methodology and tooling for detecting when Fortran `bind(C)` exports and Zig `extern fn` declarations fall out of sync. This is the most common source of linker errors in Zortran projects.

### The Sync Problem

```
Fortran adds:    fm_new_kernel(...) bind(C, name="fm_new_kernel")
Zig has NOT:     extern fn fm_new_kernel(...)  ← MISSING

Result: Zig code can't call the new kernel

Fortran removes: fm_old_kernel  (deleted)
Zig still has:   extern fn fm_old_kernel(...)  ← STALE

Result: Linker error "undefined reference to fm_old_kernel"
```

---

## Symbol Inventory

### Extract Fortran Exports (from .a archive)

```bash
# List all exported symbols (T = text/code section)
nm -g libforcore.a | grep " T " | awk '{print $3}' | sort

# Example output:
# fm_add
# fm_compute
# fm_dot
# fm_fft_forward
# fm_mat_inverse
# fm_quat_multiply
# fm_rigid_body_step
# fm_vec3_normalize
```

### Extract Zig Declarations (from fortran.zig)

```bash
# Parse extern fn names from fortran.zig
grep -oP 'pub extern fn \K[a-z_0-9]+' src/zig/fortran.zig | sort

# Or with Zig's AST (more robust):
# zig ast-check src/zig/fortran.zig  (validates syntax)
```

### Extract Fortran bind(C) Names (from .f90 source)

```bash
# From source (before compilation)
grep -oP 'name="[^"]+' src/fortran/*.f90 | grep -oP '"[^"]+' | tr -d '"' | sort
```

---

## Drift Detection

### Diff-Based Check

```bash
#!/bin/bash
# check_symbol_sync.sh

ARCHIVE="zig-out/lib/libforcore.a"
FORTRAN_ZIG="src/zig/fortran.zig"

# Extract symbols from archive
nm -g "$ARCHIVE" | grep " T " | awk '{print $3}' | sort > /tmp/fortran_exports.txt

# Extract extern fn names from Zig
grep -oP 'pub extern fn \K[a-z_0-9]+' "$FORTRAN_ZIG" | sort > /tmp/zig_externs.txt

# Compare
echo "=== In Fortran but NOT in Zig (missing wrappers) ==="
comm -23 /tmp/fortran_exports.txt /tmp/zig_externs.txt

echo "=== In Zig but NOT in Fortran (stale declarations) ==="
comm -13 /tmp/fortran_exports.txt /tmp/zig_externs.txt

# Count
FORTRAN_COUNT=$(wc -l < /tmp/fortran_exports.txt)
ZIG_COUNT=$(wc -l < /tmp/zig_externs.txt)
echo "Fortran exports: $FORTRAN_COUNT"
echo "Zig externs: $ZIG_COUNT"

if ! diff -q /tmp/fortran_exports.txt /tmp/zig_externs.txt > /dev/null 2>&1; then
    echo "SYNC FAILURE: Symbol mismatch detected"
    exit 1
fi
echo "SYNC OK: All symbols match"
```

---

## Automation

### build.zig Sync Check Step

```zig
const sync_check = b.addSystemCommand(&.{
    "bash", "scripts/check_symbol_sync.sh",
});
sync_check.step.dependOn(&ar_cmd.step);

const sync_step = b.step("sync-check", "Verify Fortran↔Zig symbol sync");
sync_step.dependOn(&sync_check.step);

// Make test depend on sync check
tests.step.dependOn(&sync_check.step);
```

### Symbol Count Header Comment

```zig
// src/zig/fortran.zig
// Symbol count: 23 extern fn declarations
// Last sync: 2026-03-15
// Sync command: zig build sync-check
```

---

## CI Integration

### GitHub Actions

```yaml
- name: Check Fortran↔Zig symbol sync
  run: |
    zig build  # Build Fortran archives first
    zig build sync-check
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Only run if Fortran or Zig files changed
if git diff --cached --name-only | grep -qE '\.(f90|zig)$'; then
    zig build sync-check || {
        echo "Symbol sync check failed. Run 'zig build sync-check' for details."
        exit 1
    }
fi
```

---

## Workflows

### After Adding a Fortran Kernel

1. Write Fortran function with `bind(C, name="fm_...")`
2. Rebuild archive: `zig build`
3. Run sync check: `zig build sync-check` → shows "missing wrapper"
4. Add `extern fn` to `fortran.zig`
5. Add safe wrapper to `lib.zig`
6. Run sync check again → should pass
7. Write tests, run `zig build test`

### After Removing a Fortran Kernel

1. Remove Fortran function
2. Rebuild archive
3. Run sync check → shows "stale declaration"
4. Remove `extern fn` from `fortran.zig`
5. Remove safe wrapper from `lib.zig`
6. Remove tests
7. Run sync check → pass

---

## Anti-Patterns

- **Manual counting** — automate with scripts, don't eyeball it
- **Ignoring sync failures** — fix immediately, don't let drift accumulate
- **Checking archives only** — also verify against `.f90` source for name= values
- **No CI gate** — sync check must block merge

---

## Cross-References

- [zortran-bind-c](../zortran-bind-c/SKILL.md) — Fortran exports being tracked
- [zortran-zig-wrapper](../zortran-zig-wrapper/SKILL.md) — Zig declarations being tracked
- [zortran-scaffold](../zortran-scaffold/SKILL.md) — Build system setup
