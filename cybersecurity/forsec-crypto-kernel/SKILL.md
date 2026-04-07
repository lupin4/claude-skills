---
name: "forsec-crypto-kernel"
description: "Use when implementing Block cipher structure (Feistel, SPN), hash kernels (Merkle-Damgard), HMAC, constant-time arithmetic, key material handling, zeroization. Defensive security context."
---

# forSec Crypto Kernel

Block cipher structure (Feistel, SPN), hash kernels (Merkle-Damgard), HMAC, constant-time arithmetic, key material handling, zeroization. All implementations in Fortran+Zig from scratch — not wrapping OpenSSL. Authorized defensive security and research context only.

---

## Overview

### What This Skill Does

Part of the forSec ecosystem. Implements security primitives and patterns in Fortran (constant-time numerical operations) and Zig (systems-level security, protocol handling, sandboxing).

## Block Cipher Structure

### Feistel Network (e.g., DES-like)

```fortran
subroutine fm_feistel_round(left, right, round_key, n_bytes) &
    bind(C, name="fm_feistel_round")
  integer(c_int), intent(in), value :: n_bytes
  integer(c_int8_t), intent(inout) :: left(n_bytes), right(n_bytes)
  integer(c_int8_t), intent(in) :: round_key(n_bytes)
  integer(c_int8_t) :: f_output(n_bytes), temp(n_bytes)
  ! temp = right
  temp = right
  ! right = left XOR F(right, key)
  call round_function(right, round_key, f_output, n_bytes)
  right = ieor(left, f_output)  ! XOR
  ! left = temp
  left = temp
end subroutine
```

### Zeroization Pattern

```zig
fn zeroize(buf: []u8) void {
    @memset(buf, 0);
    // Compiler barrier to prevent optimization
    std.mem.doNotOptimizeAway(buf.ptr);
}

// Usage with errdefer
fn processKey(key: []const u8) !void {
    var key_copy: [32]u8 = undefined;
    @memcpy(&key_copy, key[0..32]);
    defer zeroize(&key_copy);  // Always zero key material
    // ... use key_copy ...
}
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
