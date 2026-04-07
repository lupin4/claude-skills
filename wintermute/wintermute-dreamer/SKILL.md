---
name: "wintermute-dreamer"
description: "Use when designing or implementing Sleep-learning, autoresearch loop (query→search→synthesis→memory), Karpathy pattern (generate→score→filter), cost budgeting."
---

# Wintermute Dreamer

Sleep-learning, autoresearch loop (query→search→synthesis→memory), Karpathy pattern (generate→score→filter), cost budgeting. Wintermute is a from-scratch agent framework — NOT built on LangChain or similar.

---

## Overview

## Autoresearch Loop

```
activate (idle detected OR scheduled)
  → generate_queries(context, gaps)
    → search(queries, sources)
      → synthesize(results, prior_knowledge)
        → commit_to_memory(synthesis)
          → sleep (wait for next activation)
```

## Karpathy Pattern Integration

"Learn by generating, scoring, filtering":

1. **Generate:** Produce candidate responses/solutions
2. **Score:** Rate each candidate against quality criteria
3. **Filter:** Keep top-k, discard rest
4. **Distill:** Extract patterns from top candidates → memory

## Cost Budgeting

```zig
const DreamerBudget = struct {
    max_queries_per_session: u32 = 10,
    max_tokens_per_session: u64 = 50_000,
    max_cost_usd: f64 = 0.50,
    cooldown_minutes: u32 = 30,
    
    tokens_used: u64 = 0,
    
    fn canContinue(self: DreamerBudget) bool {
        return self.tokens_used < self.max_tokens_per_session;
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
