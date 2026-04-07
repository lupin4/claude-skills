# Comptime Patterns Catalog

## Pattern 1: Type-Based Dispatch
Select implementation based on numeric type (f32 vs f64).

## Pattern 2: Algorithm Family Switch
Select between BLAS, custom, or fallback implementations.

## Pattern 3: Feature Detection
Enable/disable features based on target CPU capabilities.

```zig
const has_avx = std.Target.x86.featureSetHas(target.cpu.features, .avx2);
if (has_avx) { ... }
```

## Pattern 4: Comptime String Building
Generate extern fn names at compile time.

```zig
fn getKernelName(comptime prefix: []const u8, comptime suffix: []const u8) [:0]const u8 {
    return prefix ++ "_" ++ suffix;
}
```
