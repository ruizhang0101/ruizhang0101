---
name: KVFlow (Jul 2025) — workflow-aware prefix cache with statically-declared Agent Step Graph
type: research
scope: area:kv-cache-agent-serving
---

**KVFlow** — Pan, Patel, Hu, Shen, Guan, Li, Qin, Wang, Ding (UCSD + AWS), arXiv 2507.07400, Jul 2025.

Workflow-aware prefix cache for multi-agent serving. Users declare workflow via SGLang `sgl.function` annotations. Each agent gets a step-aggregation function (`max(E1, E2)+1` for AND-deps, `min(E1, E2)+1` for OR-deps); steps-to-execution propagates through the radix tree. Eviction priority = steps-to-execution at last-fixed-prompt KV node, propagated upward; min-priority at shared nodes; varying suffixes always evict first. Prefetch loads next-step agents in background; for branching, hedges by prefetching all candidates within a concurrency limit. Status-aware scheduling skips a request if its prefetch hasn't completed. Built on SGLang + HiCache.

**Numbers:**
- Synthetic 10-agent sequential workflow at 8192/32/32 on Llama-3.1-8B/A10G → 1.83× over HiCache, 2.91× over plain SGLang.
- High-concurrency synthetic → up to 2.19× over HiCache.
- Realistic PEER-style 4-agent Financial QA → 1.12× / 1.08× (much smaller — declared workflow does less work when the agents are realistic).
- No ablation isolating eviction-only vs prefetch-only contributions.

**Key constraint:** the workflow is statically declared by the user. Cannot handle dynamic LLM-selected routing (AutoGen `SelectorGroupChat`-style) without manual graph specification per session.

**Branching strategy:** hedges by prefetching all candidates — wasteful at high branching factor; only viable when fanout is small.

**Why it's relevant for KV-cache-for-agents work:**
Closest direct competitor to any "predict-then-prefetch for multi-agent serving" approach. Critical lever for differentiation: **learned vs declared** workflow knowledge. Match their evaluation surface (≥2 hardware, ≥2 models, both synthetic + realistic agent workloads).
