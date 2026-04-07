# Test Patterns Catalog

## Pattern 1: Known-Answer Test (KAT)

Test against pre-computed correct answers.

```zig
test "KAT: fm_det_3x3" {
    const mat = [9]f64{ 1, 2, 3, 0, 1, 4, 5, 6, 0 };
    const expected_det = 1.0;
    var result: f64 = undefined;
    fortran.fm_det_3x3(&mat, &result);
    try testing.expectApproxEqAbs(result, expected_det, 1e-14);
}
```

## Pattern 2: Property-Based Test

Verify mathematical properties without knowing the exact answer.

```zig
test "property: inverse(A) * A = I" {
    const a = [4]f64{ 2, 1, 1, 3 };  // 2x2
    var a_inv: [4]f64 = undefined;
    var product: [4]f64 = undefined;
    fortran.fm_mat_inverse(&a, &a_inv, 2, &status);
    fortran.fm_matmul(&a_inv, &a, &product, 2, 2, 2, &status);
    // Check product ≈ identity
    try testing.expectApproxEqAbs(product[0], 1.0, 1e-12);
    try testing.expectApproxEqAbs(product[3], 1.0, 1e-12);
}

test "property: quaternion multiply preserves unit norm" {
    const q1 = [4]f64{ 0.5, 0.5, 0.5, 0.5 };
    const q2 = [4]f64{ 0.707, 0.0, 0.707, 0.0 };
    var result: [4]f64 = undefined;
    fortran.fm_quat_multiply(&q1, &q2, &result);
    const norm = @sqrt(result[0]*result[0] + result[1]*result[1] + result[2]*result[2] + result[3]*result[3]);
    try testing.expectApproxEqAbs(norm, 1.0, 1e-12);
}
```

## Pattern 3: Symmetry Test

Verify symmetry properties of operations.

```zig
test "symmetry: dot(a,b) == dot(b,a)" {
    const a = [_]f64{ 1.0, 2.0, 3.0 };
    const b = [_]f64{ 4.0, 5.0, 6.0 };
    const ab = try lib.dot(&a, &b);
    const ba = try lib.dot(&b, &a);
    try testing.expectApproxEqAbs(ab, ba, 1e-15);
}
```

## Pattern 4: Boundary/Edge Case Test

```zig
test "edge: fm_solve with singular matrix returns error" {
    const singular = [4]f64{ 1, 2, 2, 4 }; // det = 0
    var x: [2]f64 = undefined;
    const b = [2]f64{ 1, 1 };
    const result = lib.solve(&singular, &b, &x, 2);
    try testing.expectError(error.SingularMatrix, result);
}

test "edge: fm_fft with n=1 is identity" {
    var data = [1]f64{42.0};
    var status: c_int = 0;
    fortran.fm_fft_forward(data[0..].ptr, 1, &status);
    try testing.expectEqual(status, 0);
    try testing.expectApproxEqAbs(data[0], 42.0, 1e-15);
}
```

## Pattern 5: Convergence Test

For iterative algorithms, verify convergence rate.

```zig
test "convergence: fm_newton_solve converges quadratically" {
    var errors: [10]f64 = undefined;
    var n_iters: c_int = 0;
    fortran.fm_newton_solve(x0, &errors, &n_iters, &status);
    // Quadratic convergence: error[i+1] ≈ C * error[i]^2
    for (1..@intCast(n_iters)) |i| {
        const ratio = errors[i] / (errors[i-1] * errors[i-1]);
        try testing.expect(ratio < 10.0); // bounded convergence constant
    }
}
```

## Assertion Helpers

```zig
/// Check two float arrays are approximately equal
fn expectArrayApproxEq(a: []const f64, b: []const f64, tol: f64) !void {
    try testing.expectEqual(a.len, b.len);
    for (a, b, 0..) |av, bv, i| {
        testing.expectApproxEqAbs(av, bv, tol) catch {
            std.debug.print("index {}: {} vs {} (diff {e})\n", .{i, av, bv, @abs(av-bv)});
            return error.TestExpectedApproxEqAbs;
        };
    }
}

/// Check a matrix is approximately symmetric
fn expectSymmetric(mat: []const f64, n: usize, tol: f64) !void {
    for (0..n) |i| {
        for (i+1..n) |j| {
            try testing.expectApproxEqAbs(mat[i*n+j], mat[j*n+i], tol);
        }
    }
}

/// Check a matrix is approximately orthogonal (Q^T Q ≈ I)
fn expectOrthogonal(q: []const f64, n: usize, tol: f64) !void {
    for (0..n) |i| {
        for (0..n) |j| {
            var dot: f64 = 0;
            for (0..n) |k| dot += q[k*n+i] * q[k*n+j];
            const expected: f64 = if (i == j) 1.0 else 0.0;
            try testing.expectApproxEqAbs(dot, expected, tol);
        }
    }
}
```
