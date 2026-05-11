---
name: InfiniGen (OSDI'24) — within-iteration KV prefetch via SVD-skewed Q/K
type: research
scope: area:kv-cache-agent-serving
---

**InfiniGen** — Lee, Lee, Seo, Sim (Seoul National University), OSDI'24, arXiv 2406.19707.

Speculative within-iteration KV prefetch. Multiplies W_Q and W_K offline by an orthogonal matrix V from SVD(Q) — preserves QK^T exactly while concentrating magnitude into a few outlier columns. At layer i−1, uses that layer's attention input + 30% partial weights / key cache of layer i to compute a *speculated* layer-i attention score, then prefetches KV entries scoring above (max − α). Threshold α=4 keeps tokens above ~e^−4 ≈ 1.8% post-softmax importance. Reports 3× speedup over FlexGen+H2O on OPT-13B at 2048 seq len. Uses FlexGen as base (not vLLM).

**Axis:** layer-to-layer within one inference step (vertical). Orthogonal to cross-request prefix reuse (horizontal).

**Why it's relevant for KV-cache-for-agents work:**
This is the published prior art for "prefetching KV cache." Any new prefetching mechanism cannot claim "first prefetching" — only narrower variants (cross-session, learned, etc.).

The α-threshold formulation is mathematically cleaner than ad-hoc absolute thresholds (`p_min=0.10`-style) — it's relative importance e^−α with a tunable α.

The evaluation breadth template (5 models × 5 lm-eval tasks × 4 KV sizes, perplexity at scaling sequence length) is a useful reference target.
