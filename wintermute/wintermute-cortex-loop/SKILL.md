---
name: "wintermute-cortex-loop"
description: "Use when designing or implementing 3ms cortex loop (sense→decide→act), priority binding tiers (P0 reflexive, P1 reactive, P2 deliberative, P3 background), subagent lifecycle."
---

# Wintermute Cortex Loop

3ms cortex loop (sense→decide→act), priority binding tiers (P0 reflexive, P1 reactive, P2 deliberative, P3 background), subagent lifecycle. Wintermute is a from-scratch agent framework — NOT built on LangChain or similar.

---

## Overview

## 3ms Tick Budget

```
Total budget: 3ms per tick
├── Sense   (0.5ms) — Read inputs, check message queues
├── Decide  (1.5ms) — Priority-based action selection
├── Act     (0.5ms) — Execute chosen action
└── Reserve (0.5ms) — Subagent management, metrics
```

## Priority Binding Tiers

| Tier | Latency | Scope | Examples |
|------|---------|-------|---------|
| P0 (reflexive) | < 1ms | Pre-computed responses | Safety stops, cached replies |
| P1 (reactive) | < 3ms | Single-step reasoning | Tool calls, simple queries |
| P2 (deliberative) | Async | Multi-step reasoning | Planning, research, analysis |
| P3 (background) | Idle | Maintenance tasks | Memory consolidation, dreamer |

## Subagent Lifecycle

```
spawn(task, capabilities, budget)
  → running(progress, heartbeat)
    → collect(result) | timeout | error
      → terminate(cleanup)
```

```zig
const Subagent = struct {
    id: u64,
    task: []const u8,
    capabilities: Capability,
    budget_ms: u64,
    state: enum { spawning, running, collecting, terminated },
    spawn_time: i64,
    
    fn isOverBudget(self: Subagent) bool {
        return std.time.timestamp() - self.spawn_time > self.budget_ms;
    }
};
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
