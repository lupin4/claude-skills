---
name: "zortran-test-harness"
description: "Use when writing Zig test blocks to validate Fortran kernels. Covers test organization, floating-point tolerance comparisons, reference data loading, and kernel family test patterns."
---

# Zortran Test Harness

Zig test blocks for validating Fortran numerical kernels. Tests call Fortran directly through the `extern fn` layer, compare against reference data with configurable tolerances, and are organized by kernel family.

---

## Table of Contents

- [Overview](#overview)
- [Test Block Structure](#test-block-structure)
- [Floating-Point Tolerance](#floating-point-tolerance)
- [Reference Data Testing](#reference-data-testing)
- [Kernel Family Organization](#kernel-family-organization)
- [Workflows](#workflows)
- [Anti-Patterns](#anti-patterns)
- [Cross-References](#cross-references)

---

## Overview

### What This Skill Does

Provides the testing patterns for validating Fortran numerical kernels from Zig. Fortran has no native test framework that integrates with Zig's build system, so all kernel tests are Zig `test` blocks that call through the `extern fn` interface.

### Prerequisites

- Fortran archives built and linked (see zortran-scaffold)
- `fortran.zig` with extern declarations (see zortran-zig-wrapper)

---

## Test Block Structure

```zig
const std = @import("std");
const fortran = @import("../src/zig/fortran.zig");
const lib = @import("../src/zig/lib.zig");
const testing = std.testing;

test "fm_dot computes dot product correctly" {
    const x = [_]f64{ 1.0, 2.0, 3.0 };
    const y = [_]f64{ 4.0, 5.0, 6.0 };
    const result = try lib.dot(&x, &y);
    try testing.expectApproxEqAbs(result, 32.0, 1e-12);
}

test "fm_dot handles empty array" {
    const result = try lib.dot(&[_]f64{}, &[_]f64{});
    try testing.expectApproxEqAbs(result, 0.0, 1e-15);
}

test "fm_quat_multiply identity" {
    const identity = [4]f64{ 1.0, 0.0, 0.0, 0.0 }; // w, x, y, z
    const q = [4]f64{ 0.707, 0.707, 0.0, 0.0 };
    var result: [4]f64 = undefined;
    fortran.fm_quat_multiply(&identity, &q, &result);

    for (0..4) |i| {
        try testing.expectApproxEqAbs(result[i], q[i], 1e-10);
    }
}
```

---

## Floating-Point Tolerance

### Absolute Tolerance

```zig
// For values near zero or with known magnitude
try testing.expectApproxEqAbs(computed, expected, 1e-12);
```

### Relative Tolerance

```zig
// For values that may vary in magnitude
try testing.expectApproxEqRel(computed, expected, 1e-10);
```

### Tolerance Guidelines

| Kernel Type | Absolute Tol | Relative Tol | Rationale |
|------------|-------------|-------------|-----------|
| Linear algebra (BLAS) | 1e-14 | 1e-12 | Near machine epsilon |
| FFT | 1e-10 | 1e-8 | Accumulation errors |
| ODE integrators | 1e-6 | 1e-4 | Discretization error |
| Optimization | 1e-6 | 1e-4 | Convergence tolerance |
| Quantization | 1e-2 | 1e-1 | Intentional precision loss |

### Custom Tolerance Helper

```zig
fn expectArrayApproxEq(computed: []const f64, expected: []const f64, tol: f64) !void {
    try testing.expectEqual(computed.len, expected.len);
    for (computed, expected, 0..) |c, e, i| {
        testing.expectApproxEqAbs(c, e, tol) catch |err| {
            std.debug.print("Mismatch at index {}: computed={d:.15}, expected={d:.15}, diff={d:.2e}\n", .{ i, c, e, @abs(c - e) });
            return err;
        };
    }
}
```

---

## Reference Data Testing

### Embedding Test Data

```zig
// Small reference datasets can be embedded directly
const reference_3x3 = [9]f64{
    1.0, 0.0, 0.0,
    0.0, 1.0, 0.0,
    0.0, 0.0, 1.0,
};

test "fm_mat_inverse 3x3 identity" {
    var result: [9]f64 = undefined;
    var status: c_int = 0;
    fortran.fm_mat_inverse(&reference_3x3, &result, 3, &status);
    try testing.expectEqual(status, 0);
    try expectArrayApproxEq(&result, &reference_3x3, 1e-15);
}
```

### Loading Test Data from Files

```zig
test "fm_fft matches MATLAB reference" {
    // Load reference from test_data/fft_reference.bin
    const data = @embedFile("../tests/test_data/fft_reference.bin");
    const floats = std.mem.bytesAsSlice(f64, data);
    const n = floats.len / 2;
    const input = floats[0..n];
    const expected_output = floats[n..];

    var output = try testing.allocator.alloc(f64, n);
    defer testing.allocator.free(output);

    @memcpy(output, input);
    var status: c_int = 0;
    fortran.fm_fft_forward(output.ptr, @intCast(n), &status);
    try testing.expectEqual(status, 0);
    try expectArrayApproxEq(output, expected_output, 1e-10);
}
```

---

## Kernel Family Organization

```
tests/
├── test_linear_algebra.zig    # BLAS, LAPACK wrappers
├── test_fft.zig               # FFT kernel family
├── test_quaternion.zig        # Lie group / rotation kernels
├── test_rigid_body.zig        # Physics simulation kernels
├── test_quantization.zig      # ML quantization kernels
└── test_data/
    ├── fft_reference.bin
    ├── svd_reference.bin
    └── README.md              # How reference data was generated
```

---

## Workflows

### Writing a New Kernel Test

1. Identify the kernel function and its Fortran signature
2. Determine the appropriate tolerance (see guidelines table)
3. Write test with known-answer input/output pair
4. Add edge case tests (zero input, empty array, singular matrix)
5. Run `zig build test` and verify pass

### Generating Reference Data

1. Implement reference computation in Python/MATLAB
2. Export as binary (f64 array) or CSV
3. Place in `tests/test_data/`
4. Document generation method in `test_data/README.md`
5. Use `@embedFile` to load in Zig tests

---

## Anti-Patterns

- **Exact equality for floats** — always use tolerance comparison
- **Testing only happy path** — test edge cases: empty, singular, NaN, huge values
- **Hardcoded magic numbers** — name your test data, comment expected values
- **Testing Zig wrappers only** — also test raw `extern fn` to catch ABI issues
- **Skipping reference data docs** — always document how reference data was generated

---

## Cross-References

- [zortran-bind-c](../zortran-bind-c/SKILL.md) — The Fortran exports being tested
- [zortran-zig-wrapper](../zortran-zig-wrapper/SKILL.md) — Safe wrappers being tested
- [zig-build-system](../../zig-systems/zig-build-system/SKILL.md) — Test step in build.zig
