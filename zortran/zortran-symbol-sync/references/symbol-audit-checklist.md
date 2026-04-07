# Symbol Audit Checklist

## Pre-Merge Audit

- [ ] `zig build sync-check` passes (zero drift)
- [ ] `fortran.zig` header comment has correct symbol count
- [ ] Every new `extern fn` has a corresponding safe wrapper in `lib.zig`
- [ ] Every removed `extern fn` has no remaining callers
- [ ] All `bind(C, name="...")` values follow `fm_`/`ft_`/`fs_` conventions
- [ ] No duplicate symbol names across archives

## Quarterly Audit

- [ ] Run `nm -g` on all `.a` archives, compile master symbol list
- [ ] Cross-reference with all `fortran.zig` files across repos
- [ ] Check for symbol name collisions between sibling repos
- [ ] Verify DEPENDENCY_RULES.md matches actual link dependencies
- [ ] Update symbol count in all `fortran.zig` header comments

## Common Sync Failures

| Symptom | Cause | Fix |
|---------|-------|-----|
| `undefined reference to fm_*` | Zig declares it, Fortran doesn't export it | Remove stale `extern fn` or add missing Fortran export |
| No error but kernel not callable | Forgot to add `extern fn` | Add declaration to `fortran.zig` |
| Link succeeds but wrong results | `bind(C, name=)` typo — links to wrong function | Verify name= matches exactly |
| `multiple definition` | Same symbol in two archives | Remove duplicate, check DEPENDENCY_RULES |
| Works on Linux, fails on macOS | ar format mismatch | Use consistent ar binary (see archive-tiers) |

## Tools Quick Reference

```bash
# Exported symbols from archive
nm -g lib.a | grep " T "

# Undefined (needed) symbols
nm -u object.o

# All symbols with addresses
nm -n lib.a

# Fortran bind(C) names from source
grep -rn 'bind(C' src/fortran/ | grep 'name='

# Zig extern fn count
grep -c 'pub extern fn' src/zig/fortran.zig

# Objdump alternative (more detail)
objdump -t lib.a | grep "g.*F"
```
