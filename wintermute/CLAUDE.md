# Wintermute Agent Architecture — Claude Code Guidance

Agent architecture skills for Wintermute — a from-scratch agent framework with a 3ms cortex loop, dreamer subsystem, CLI plugin layer, and multi-platform gateway. NOT built on LangChain or similar frameworks.

## Skills Overview (6)

| Skill | Focus |
|-------|-------|
| **wintermute-cortex-loop** | 3ms tick loop, priority binding tiers (P0-P3), subagent spawn |
| **wintermute-dreamer** | Sleep-learning, autoresearch, Karpathy pattern integration |
| **cli-anything-harness** | Stateless CLI plugin layer, JSON stdin/stdout contract |
| **foragent-state-machine** | Persistent stateful agent framework patterns |
| **multiagent-boundaries** | Orchestrator/subagent interfaces, DEPENDENCY_RULES for agents |
| **wintermolt-gateway** | OpenAI-compatible endpoint, 18-platform integration |

## Key Conventions

- **Cortex loop budget:** 3ms total (sense → decide → act)
- **Priority tiers:** P0 reflexive (< 1ms), P1 reactive (< 3ms), P2 deliberative (async), P3 background (idle)
- **Plugin contract:** JSON on stdin → JSON on stdout, exit code semantics, stateless
- **Gateway:** OpenAI API compatible (/v1/chat/completions), SSE streaming
- **State:** SQLite for persistent agent state, in-memory for ephemeral

## Cross-References

- cybersecurity/forsec-agent-sandbox — Sandboxing for subagent tool execution
- cybersecurity/forsec-audit-trail — Tamper-evident logging for agent actions
- zig-systems/ — Build system and memory patterns for agent runtime
