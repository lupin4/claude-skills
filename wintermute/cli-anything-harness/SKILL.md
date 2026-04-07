---
name: "cli-anything-harness"
description: "Use when designing or implementing Stateless CLI plugin conventions, JSON stdin/stdout contract, plugin discovery, argument schema declaration, timeout and resource limits."
---

# CLI-Anything Harness

Stateless CLI plugin conventions, JSON stdin/stdout contract, plugin discovery, argument schema declaration, timeout and resource limits. Wintermute is a from-scratch agent framework — NOT built on LangChain or similar.

---

## Overview

## Plugin Interface Contract

```
stdin  → JSON request  → Plugin executes → JSON response → stdout
                                                          → exit code
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Plugin error (see stderr) |
| 2 | Invalid input |
| 3 | Timeout |
| 124 | Resource limit exceeded |

### Request Schema

```json
{
  "action": "execute",
  "tool": "calculator",
  "args": {"expression": "2 + 3"},
  "timeout_ms": 5000,
  "capabilities": ["read_fs"]
}
```

### Response Schema

```json
{
  "status": "success",
  "result": {"value": 5},
  "metadata": {"duration_ms": 12, "tokens_used": 0}
}
```

## Plugin Discovery

```
~/.wintermute/plugins/
├── calculator/
│   ├── plugin.json    # Schema: name, version, tools[], capabilities[]
│   └── run.sh         # Entrypoint (or binary)
├── web-search/
│   ├── plugin.json
│   └── search
└── ...
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
