---
name: "multiagent-boundaries"
description: "Use when designing or implementing Orchestrator/subagent interface design, boundary enforcement, DEPENDENCY_RULES analog for agents, deadlock prevention."
---

# Multiagent Boundaries

Orchestrator/subagent interface design, boundary enforcement, DEPENDENCY_RULES analog for agents, deadlock prevention. Wintermute is a from-scratch agent framework — NOT built on LangChain or similar.

---

## Overview

## Orchestrator/Subagent Interface

```
Orchestrator
├── decompose(task) → subtasks[]
├── assign(subtask, agent) → handle
├── monitor(handle) → status
├── collect(handle) → result
└── aggregate(results[]) → final_output
```

## DEPENDENCY_RULES for Agents

```markdown
# Agent Dependency Rules

| Agent | May Call | May NOT Call |
|-------|---------|-------------|
| orchestrator | any subagent | — |
| researcher | web-search, file-reader | code-executor |
| code-executor | — (leaf agent) | anything |
| reviewer | file-reader | code-executor |
```

## Boundary Enforcement

```zig
const BoundaryPolicy = struct {
    agent_name: []const u8,
    allowed_tools: []const []const u8,
    allowed_agents: []const []const u8,
    max_depth: u32 = 3,  // Prevent infinite subagent chains
    
    fn canCall(self: BoundaryPolicy, target: []const u8) bool {
        return std.mem.indexOf([]const u8, self.allowed_agents, target) != null;
    }
};
```

## Deadlock Prevention

1. **No circular agent calls** — enforced by DEPENDENCY_RULES DAG
2. **Timeout on all calls** — every subagent has a budget
3. **Max depth** — limit subagent spawn depth (default: 3)

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
