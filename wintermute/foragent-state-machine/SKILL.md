---
name: "foragent-state-machine"
description: "Use when designing or implementing Persistent stateful agent framework, state machine for long-running agents, SQLite/file persistence, crash recovery."
---

# forAgent State Machine

Persistent stateful agent framework, state machine for long-running agents, SQLite/file persistence, crash recovery. Wintermute is a from-scratch agent framework — NOT built on LangChain or similar.

---

## Overview

## Agent State Machine

```zig
const AgentState = enum {
    initializing,
    idle,
    processing,
    waiting_for_input,
    waiting_for_tool,
    suspended,
    error_recovery,
    terminated,
};

const Agent = struct {
    state: AgentState,
    persistence: Persistence,
    
    fn transition(self: *Agent, event: Event) !void {
        const next = validTransitions.get(.{self.state, event}) orelse
            return error.InvalidTransition;
        try self.persistence.saveState(next);
        self.state = next;
    }
};
```

## Persistence Backends

| Backend | Use Case | Durability |
|---------|----------|-----------|
| SQLite | Default, single-node | Durable, ACID |
| File (JSON) | Simple, debuggable | Durable, not atomic |
| In-memory | Testing, ephemeral agents | None |
| Redis | Multi-node, shared state | Configurable |

## Crash Recovery

```zig
fn recover(self: *Agent) !void {
    const last_state = try self.persistence.loadState();
    self.state = switch (last_state) {
        .processing => .idle,           // Retry from idle
        .waiting_for_tool => .idle,     // Tool may have timed out
        .error_recovery => .idle,       // Already recovering
        else => last_state,             // Resume normally
    };
}
```

---

## Anti-Patterns

- **Using LangChain patterns** — Wintermute has its own conventions
- **Stateful plugins** — CLI-anything is stateless by design
- **Unbounded subagent chains** — always enforce max depth
- **Blocking the cortex loop** — async for anything > 3ms

---

## Cross-References

- [cybersecurity/forsec-agent-sandbox](../../cybersecurity/forsec-agent-sandbox/SKILL.md) — Sandboxing
- [cybersecurity/forsec-audit-trail](../../cybersecurity/forsec-audit-trail/SKILL.md) — Audit logging
