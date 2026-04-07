---
name: "forsec-agent-sandbox"
description: "Use when implementing Capability-based security, seccomp-bpf sandboxing, filesystem namespace isolation, network restriction, resource limits for AI agent execution. Defensive security context."
---

# forSec Agent Sandbox

Capability-based security, seccomp-bpf sandboxing, filesystem namespace isolation, network restriction, resource limits for AI agent execution. All implementations in Fortran+Zig from scratch — not wrapping OpenSSL. Authorized defensive security and research context only.

---

## Overview

### What This Skill Does

Part of the forSec ecosystem. Implements security primitives and patterns in Fortran (constant-time numerical operations) and Zig (systems-level security, protocol handling, sandboxing).

## Capability-Based Security Model

```zig
const Capability = packed struct {
    read_fs: bool = false,
    write_fs: bool = false,
    network: bool = false,
    exec: bool = false,
    gpu: bool = false,
    ipc: bool = false,
};

// Default: minimal capabilities
const sandbox_default = Capability{
    .read_fs = true,  // Read-only filesystem
};

// Tool execution: restricted capabilities
const tool_caps = Capability{
    .read_fs = true,
    .write_fs = true,  // Only to sandbox dir
};
```

## seccomp-bpf (Linux)

```zig
// Restrict syscalls for sandboxed tool execution
const allowed_syscalls = [_]u32{
    std.os.linux.SYS.read,
    std.os.linux.SYS.write,
    std.os.linux.SYS.exit,
    std.os.linux.SYS.exit_group,
    std.os.linux.SYS.brk,
    std.os.linux.SYS.mmap,
    std.os.linux.SYS.munmap,
    // NO: execve, socket, fork, clone
};
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
