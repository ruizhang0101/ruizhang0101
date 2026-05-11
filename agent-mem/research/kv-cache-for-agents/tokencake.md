---
name: Tokencake (Oct 2025) — KV-cache-centric multi-agent serving with space + time scheduling
type: research
scope: area:kv-cache-agent-serving
---

**Tokencake** — Bian, Wu, Ma, Zhuo, arXiv 2510.18586.

Two-component framework for multi-agent serving:

1. **Space Scheduler** — dynamic memory partitioning, isolating critical agents from contention so their KV doesn't get evicted by other workloads.
2. **Time Scheduler** — during external function-call stalls (when an agent waits for a tool to return), proactively offloads its idle KV from GPU and predictively reloads it before the tool returns.

**Numbers:** >47% latency reduction and up to 16.9% memory utilization gain vs vLLM on multi-agent benchmarks.

**Axis:** exploits *idle time* during tool calls within one program. Orthogonal to cross-session reuse.

**Why it's relevant for KV-cache-for-agents work:**
Supports the broader framing that "agent-aware KV management" is productive. The Space Scheduler partially overlaps with agent-aware framings but is a *partitioning* mechanism, not a *prediction* mechanism. Tool-call-stall optimization is its niche — don't try to compete there; cite as orthogonal contribution. The 47%+ headline number is heavy-tool-call regime only; cite with context.
