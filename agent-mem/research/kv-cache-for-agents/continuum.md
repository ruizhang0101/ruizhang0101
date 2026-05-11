---
name: Continuum (Nov 2025) — TTL-based KV pinning for ReAct tool-calling agents
type: research
scope: area:kv-cache-agent-serving
---

**Continuum** — Li, Mang, He, Zhang, Mao, Chen, Zhou, Cheung, Gonzalez, Stoica (UC Berkeley + Stanford + Tsinghua), arXiv 2511.02230v3, Jan 2026.

TTL-based KV cache pinning for ReAct-style tool-calling agents. When an LLM request ends with a tool call (instead of evicting per vLLM default), pin its KV cache for τ seconds. Computes optimal τ via cost-benefit:

- **Cost** = `MemUsage(r)/M × τ` (opportunity cost of blocking GPU memory).
- **Benefit** = `P(τ, f) × (CacheMissCost + OutOfOrderCost)` where:
  - `P(τ, f)` = empirical CDF probability tool f finishes within τ (learned per-tool from observed history).
  - `CacheMissCost` = prefill/reload cost saved.
  - `OutOfOrderCost` = `T × MemUsage × η`, where η = `−Corr(k, N−k)` is the workload's *memoryfulness factor* (1 = fixed turn count, 0 = geometric).
- Cold start: fixed default TTL from exponential prior.
- Combined with program-level FCFS: pinned-within-TTL requests get priority.

Built on vLLM + LMCache, ~1k LOC. Open source: `github.com/Hanchenli/vllm-continuum`.

**Numbers:** 1.12–3.66× delay, 1.10–3.22× throughput on SWE-Bench + BFCL across A100/B200/H100 with Llama-3.1-8B/70B. Up to 8.18× on real SWE-agent in distributed setting. Speedup scales with turn count (1.6× at 1× turns → 3.7× at 5× turns). Beats vLLM, Autellix, InferCept, SGLang, Dynamo.

**Axis:** within-program tool-call retention. Optimizes the gap between an agent's tool call going out and the response coming back — which dominates SWE-Bench latency.

**Why it's relevant for KV-cache-for-agents work:**
The most credible threat to any new agent-serving KV cache approach evaluated on SWE-Bench-style ReAct workloads — Continuum dominates that workload. The right framing is **orthogonal-axis**: Continuum optimizes within-program tool-call retention; cross-session prefix prediction (e.g. CacheSage-style) optimizes a different surface. Don't claim wins on SWE-Bench; pick a workload where cross-program sharing dominates.

Three reusable ideas from this paper:
1. **Cost-benefit utility model** for cache-management decisions (replaces ad-hoc thresholds).
2. **η memoryfulness factor** to quantify when learned prediction is informative vs not.
3. **Evaluation surface** (3 hardware × 2 models × 5 baselines × P90/P95 × ablation × turn-scaling × SSD offload) is the new methodological bar.
