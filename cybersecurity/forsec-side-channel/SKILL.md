---
name: "forsec-side-channel"
description: "Use when implementing Timing-safe comparison, constant-time patterns in Fortran, branch-free conditional select, cache-timing mitigations, compiler barriers. Defensive security context."
---

# forSec Side-Channel Resistance

Timing-safe comparison, constant-time patterns in Fortran, branch-free conditional select, cache-timing mitigations, compiler barriers. All implementations in Fortran+Zig from scratch — not wrapping OpenSSL. Authorized defensive security and research context only.

---

## Overview

### What This Skill Does

Part of the forSec ecosystem. Implements security primitives and patterns in Fortran (constant-time numerical operations) and Zig (systems-level security, protocol handling, sandboxing).

## Constant-Time Comparison

```fortran
! Timing-safe byte comparison — no early exit
function fm_ct_compare(a, b, n) result(equal) bind(C, name="fm_ct_compare")
  integer(c_int), intent(in), value :: n
  integer(c_int8_t), intent(in) :: a(n), b(n)
  logical(c_bool) :: equal
  integer(c_int8_t) :: diff
  integer :: i
  diff = 0
  do i = 1, n
    diff = ior(diff, ieor(a(i), b(i)))  ! Accumulate XOR differences
  end do
  equal = (diff == 0)
end function
```

## Branch-Free Conditional Select

```fortran
! Select a or b based on condition, without branching
! condition: 0 = select b, non-zero = select a
function fm_ct_select(a, b, condition) result(selected) bind(C, name="fm_ct_select")
  integer(c_int64_t), intent(in), value :: a, b, condition
  integer(c_int64_t) :: selected
  integer(c_int64_t) :: mask
  ! mask = -1 if condition != 0, else 0 (all bits set or clear)
  mask = -ishft(ior(condition, -condition), -63)  ! arithmetic right shift
  selected = ior(iand(a, mask), iand(b, not(mask)))
end function
```

## Compiler Barriers

```fortran
! Prevent compiler from optimizing away security-critical code
!dir$ noinline   ! Intel: prevent inlining of this routine
!GCC$ noinline   ! GCC: same
```

```zig
// Zig: prevent optimization of security operations
std.mem.doNotOptimizeAway(ptr);
@fence(.seq_cst);  // Full memory fence
```

---

## Anti-Patterns

- **Branching on secrets** — all crypto paths must be constant-time
- **Not zeroing key material** — defer zeroize on all key buffers
- **Skipping compiler barriers** — optimizers WILL remove dead stores to key buffers
- **Using OpenSSL wrappers** — forSec implements from scratch for portability

---

## Cross-References

- [zortran/](../../zortran/) — Archive tier system, build patterns
- [zig-systems/](../../zig-systems/) — Error unions, allocators
- [wintermute/](../../wintermute/) — Agent sandbox applies to Wintermute subagents
