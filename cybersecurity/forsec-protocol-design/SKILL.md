---
name: "forsec-protocol-design"
description: "Use when implementing Network protocol state machines in Zig, message framing, length-prefix protocols, TLS-inspired handshake, state transition validation. Defensive security context."
---

# forSec Protocol Design

Network protocol state machines in Zig, message framing, length-prefix protocols, TLS-inspired handshake, state transition validation. All implementations in Fortran+Zig from scratch — not wrapping OpenSSL. Authorized defensive security and research context only.

---

## Overview

### What This Skill Does

Part of the forSec ecosystem. Implements security primitives and patterns in Fortran (constant-time numerical operations) and Zig (systems-level security, protocol handling, sandboxing).

## State Machine Pattern

```zig
const ProtocolState = enum {
    idle,
    handshake_sent,
    handshake_received,
    authenticated,
    data_transfer,
    closing,
    closed,
    error_state,
};

const Protocol = struct {
    state: ProtocolState = .idle,
    
    fn handleEvent(self: *Protocol, event: Event) !void {
        switch (self.state) {
            .idle => switch (event) {
                .connect => {
                    try self.sendHandshake();
                    self.state = .handshake_sent;
                },
                else => return error.InvalidTransition,
            },
            .handshake_sent => switch (event) {
                .handshake_ack => {
                    try self.verifyHandshake(event.payload);
                    self.state = .authenticated;
                },
                .timeout => self.state = .error_state,
                else => return error.InvalidTransition,
            },
            // ... other states
        }
    }
};
```

## Message Framing

```zig
// Length-prefix framing: [4-byte length][payload]
fn readFrame(reader: anytype, buf: []u8) ![]u8 {
    var len_bytes: [4]u8 = undefined;
    try reader.readNoEof(&len_bytes);
    const len = std.mem.readInt(u32, &len_bytes, .big);
    if (len > buf.len) return error.FrameTooLarge;
    try reader.readNoEof(buf[0..len]);
    return buf[0..len];
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
