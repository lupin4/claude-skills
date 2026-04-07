---
name: "forsec-build-arch"
description: "Use when implementing Importing 18 sibling .a archives, topological link order, symbol visibility (Fortran PRIVATE/PUBLIC + Zig export), duplicate symbol resolution. Defensive security context."
---

# forSec Build Architecture

Importing 18 sibling .a archives, topological link order, symbol visibility (Fortran PRIVATE/PUBLIC + Zig export), duplicate symbol resolution. All implementations in Fortran+Zig from scratch — not wrapping OpenSSL. Authorized defensive security and research context only.

---

## Overview

### What This Skill Does

Part of the forSec ecosystem. Implements security primitives and patterns in Fortran (constant-time numerical operations) and Zig (systems-level security, protocol handling, sandboxing).

## 18-Archive Dependency Graph

forSec links 18 sibling `.a` archives. Link order must follow topological sort:

```
Tier-3 (experimental)
  ↓
Tier-2 (domain): forsec, fordsp, forml, forsim, fortorch...
  ↓
Tier-1 (core): formath, forutil, fortypes
  ↓
System: libgfortran, libm
```

### build.zig for 18 Archives

```zig
const archive_order = [_][]const u8{
    // Tier-3 first (most dependent)
    "forexp_sandbox", "forexp_audit",
    // Tier-2
    "forsec", "fordsp", "forml", "forsim", "fortorch",
    "forcv", "forfusion", "forgeo", "forgraph",
    "fornav", "forai", "forbayes", "forlearn",
    // Tier-1
    "forutil", "formath", "fortypes",
};

for (archive_order) |name| {
    lib.addLibraryPath(b.path("lib/" ++ name));
    lib.linkSystemLibrary(name);
}
```

### Symbol Visibility

```fortran
module fm_crypto_internal
  implicit none
  private  ! Nothing exported to other modules
  ! Internal implementation only
end module

module fm_crypto_api
  use fm_crypto_internal  ! Uses internal, but only exports bind(C) wrappers
  implicit none
  private
  public :: fm_aes_encrypt  ! Only this is visible externally
end module
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
