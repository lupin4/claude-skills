# Cybersecurity (forSec) — Claude Code Guidance

Security-focused skills for Fortran+Zig — crypto primitives, side-channel resistance, protocol design, agent sandboxing, and tamper-evident audit trails. All implementations from scratch, not wrapping OpenSSL.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **forsec-build-arch** | 18 sibling .a archive linking, symbol visibility, link order |
| **forsec-crypto-kernel** | Block ciphers, hash functions, HMAC, constant-time arithmetic |
| **forsec-side-channel** | Timing-safe comparison, branch-free select, compiler barriers |
| **forsec-protocol-design** | Network protocol state machines in Zig |
| **forsec-agent-sandbox** | seccomp-bpf sandboxing, capability restriction for AI agents |
| **forsec-audit-trail** | Hash-chained logging, Merkle tree verification |

## Key Conventions

- **Context:** Defensive security, research, CTF — authorized use only
- **Constant-time:** ALL crypto paths must be constant-time (no branching on secrets)
- **Zeroization:** Key material zeroed immediately after use
- **Compiler barriers:** `!dir$ noinline` (Fortran), `@fence(.seq_cst)` (Zig) to prevent optimization of security-critical code
- **Build:** 18 sibling `.a` archives linked in topological order

## Cross-References

- zortran/ — Archive tier system, build.zig patterns
- zig-systems/ — Error unions, allocator patterns for crypto buffers
- wintermute/ — Agent sandbox applies to Wintermute subagents
