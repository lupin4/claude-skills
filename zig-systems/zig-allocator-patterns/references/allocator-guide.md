# Allocator Selection Guide

## Decision Tree

1. Batch processing (allocate, compute, free all)? → **ArenaAllocator**
2. Development/testing (need leak detection)? → **GeneralPurposeAllocator**
3. Fixed memory budget (embedded)? → **FixedBufferAllocator**
4. Single large allocation? → **page_allocator**
5. Performance-critical hot path? → **FixedBufferAllocator** or pre-allocated pool

## Pipeline Memory Pattern

```
Kernel Pipeline:
  Arena 1 (input prep) → Arena 2 (compute) → Arena 3 (output)
  Each arena freed after its phase completes
```
