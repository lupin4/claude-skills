---
name: "forsec-audit-trail"
description: "Use when implementing Hash-chained log entries, Merkle tree batch verification, append-only storage, log rotation with chain continuity. Defensive security context."
---

# forSec Audit Trail

Hash-chained log entries, Merkle tree batch verification, append-only storage, log rotation with chain continuity. All implementations in Fortran+Zig from scratch — not wrapping OpenSSL. Authorized defensive security and research context only.

---

## Overview

### What This Skill Does

Part of the forSec ecosystem. Implements security primitives and patterns in Fortran (constant-time numerical operations) and Zig (systems-level security, protocol handling, sandboxing).

## Hash-Chained Log Entries

```zig
const LogEntry = struct {
    timestamp: i64,
    action: []const u8,
    actor: []const u8,
    data_hash: [32]u8,      // SHA-256 of action data
    prev_hash: [32]u8,      // Hash of previous entry
    entry_hash: [32]u8,     // Hash of this entire entry
};

fn appendEntry(chain: *Chain, action: []const u8, actor: []const u8) !void {
    var entry: LogEntry = .{
        .timestamp = std.time.timestamp(),
        .action = action,
        .actor = actor,
        .data_hash = computeHash(action),
        .prev_hash = chain.last_hash,
        .entry_hash = undefined,
    };
    // entry_hash = H(timestamp || action || actor || data_hash || prev_hash)
    entry.entry_hash = computeEntryHash(entry);
    try chain.storage.append(entry);
    chain.last_hash = entry.entry_hash;
}
```

## Tamper Detection

```zig
fn verifyChain(entries: []const LogEntry) !bool {
    for (entries[1..], 1..) |entry, i| {
        // Verify prev_hash links
        if (!std.mem.eql(u8, &entry.prev_hash, &entries[i-1].entry_hash))
            return false;
        // Verify entry_hash integrity
        if (!std.mem.eql(u8, &entry.entry_hash, &computeEntryHash(entry)))
            return false;
    }
    return true;
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
