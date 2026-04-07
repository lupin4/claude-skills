---
name: "zig-allocator-patterns"
description: "Use when choosing memory allocation strategies for Fortran+Zig kernel pipelines. Covers ArenaAllocator, GeneralPurposeAllocator, FixedBufferAllocator, and leak detection."
---

# Zig Allocator Patterns

Memory allocation strategies for kernel pipelines — choosing the right allocator for batch processing, development/debug, and embedded contexts.

---

## Allocator Selection Guide

| Allocator | Use Case | Leak Detection | Thread-Safe |
|-----------|----------|---------------|-------------|
| ArenaAllocator | Batch kernel runs (free all at once) | No | No |
| GeneralPurposeAllocator | Development/debug | Yes | Yes (configurable) |
| FixedBufferAllocator | Embedded/no-heap | No | No |
| page_allocator | Large allocations | No | Yes |

## Arena for Batch Kernel Runs

```zig
var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
defer arena.deinit();  // Frees everything at once
const allocator = arena.allocator();

// All allocations freed together when arena.deinit() runs
const workspace = try allocator.alloc(f64, n * n);
const temp = try allocator.alloc(f64, n);
// No individual free() calls needed
```

## GPA for Debug

```zig
var gpa = std.heap.GeneralPurposeAllocator(.{}){};
defer {
    const check = gpa.deinit();
    if (check == .leak) @panic("Memory leak detected");
}
const allocator = gpa.allocator();
```

---

## Anti-Patterns

- Refer to domain-specific anti-patterns in the skill content above

---

## Cross-References

- [zortran/](../../zortran/) — Fortran↔Zig interop patterns
- [fortran-kernels/](../../fortran-kernels/) — Fortran kernel conventions
