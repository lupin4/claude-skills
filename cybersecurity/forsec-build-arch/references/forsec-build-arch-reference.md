# forSec Build Architecture Reference

See SKILL.md for implementation patterns and code.

## Key Principles

1. Constant-time: no branching on secret data
2. Zeroization: key material zeroed after use
3. Compiler barriers: prevent optimization of security code
4. Defense in depth: multiple layers of protection
