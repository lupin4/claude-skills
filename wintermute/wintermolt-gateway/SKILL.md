---
name: "wintermolt-gateway"
description: "Use when designing or implementing OpenAI-compatible /v1/chat/completions, SSE streaming, platform adapter pattern for 18 platforms, auth, rate limiting."
---

# Wintermolt Gateway

OpenAI-compatible /v1/chat/completions, SSE streaming, platform adapter pattern for 18 platforms, auth, rate limiting. Wintermute is a from-scratch agent framework — NOT built on LangChain or similar.

---

## Overview

## OpenAI API Compatibility

### /v1/chat/completions

```zig
const ChatRequest = struct {
    model: []const u8,
    messages: []const Message,
    temperature: f64 = 0.7,
    max_tokens: ?u64 = null,
    stream: bool = false,
};

const ChatResponse = struct {
    id: []const u8,
    object: []const u8 = "chat.completion",
    created: i64,
    model: []const u8,
    choices: []const Choice,
    usage: Usage,
};
```

### SSE Streaming

```
data: {"id":"chatcmpl-xxx","choices":[{"delta":{"content":"Hello"}}]}

data: {"id":"chatcmpl-xxx","choices":[{"delta":{"content":" world"}}]}

data: [DONE]
```

## Platform Adapter Pattern

```zig
const PlatformAdapter = struct {
    name: []const u8,
    endpoint: []const u8,
    auth: AuthConfig,
    
    fn sendRequest(self: *PlatformAdapter, request: ChatRequest) !ChatResponse {
        // Normalize request → platform-specific format
        // Send via HTTP
        // Normalize response → OpenAI format
    }
};

// 18 adapters: OpenAI, Anthropic, Google, Mistral, Cohere,
// Ollama, vLLM, TGI, LlamaCpp, Together, Groq, Fireworks,
// Replicate, Hugging Face, Azure, AWS Bedrock, GCP Vertex, local
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
