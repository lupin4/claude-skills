# Archive Tier Guide

## ar Compatibility Matrix

| Platform | Default ar | Thin Archives | Fat Archives | Deterministic |
|----------|-----------|---------------|-------------|---------------|
| Linux (GNU) | GNU ar | `ar rcsT` | `ar rcs` | `ar rcsD` |
| macOS (system) | BSD ar | Not supported | `ar rcs` (default) | `ar -rcsS` |
| macOS (Homebrew) | GNU ar as `gar` | `gar rcsT` | `gar rcs` | `gar rcsD` |
| FreeBSD | BSD ar | Not supported | `ar rcs` | N/A |

## Tier Classification Checklist

### Tier-1 Candidates
- [ ] Used by 3+ other archives
- [ ] API stable for 6+ months
- [ ] No domain-specific logic
- [ ] Examples: math primitives, type definitions, string utilities

### Tier-2 Candidates
- [ ] Domain-specific (physics, DSP, ML, security)
- [ ] API evolves with domain needs
- [ ] Dependencies are tier-1 only
- [ ] Examples: libforsim.a, libfordsp.a, libforml.a

### Tier-3 Candidates
- [ ] Research or prototype
- [ ] API may change without notice
- [ ] May depend on tier-1 and tier-2
- [ ] Named with `forexp_` prefix
- [ ] Examples: libforexp_newquant.a, libforexp_hopfield.a

## Link Order Debugging

```bash
# List all symbols in an archive
nm -g libforcore.a | grep " T "

# Find which archive provides a symbol
for lib in lib/*.a; do
  if nm -g "$lib" 2>/dev/null | grep -q " T fm_target_func"; then
    echo "$lib provides fm_target_func"
  fi
done

# Find unresolved symbols
nm -u zig-out/obj/main.o

# Verify link order resolves all symbols
zig build 2>&1 | grep "undefined reference"
```

## Common Link Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `undefined reference to fm_*` | Wrong link order or missing archive | Reorder: dependent before dependency |
| `multiple definition of fm_*` | Duplicate symbol in two archives | Remove from one archive, verify DEPENDENCY_RULES |
| `archive member is not a valid object` | BSD/GNU ar mismatch | Use consistent ar binary |
| `cannot find -lforcore` | Missing library path | Add `addLibraryPath` in build.zig |
