---
name: KVCOMM (Oct 2025) — approximate KV reuse across non-matching prefixes via anchor pool
type: research
scope: area:kv-cache-agent-serving
---

**KVCOMM** — Ye, Gao, Ma + 8 co-authors, arXiv 2510.12872.

Training-free framework that maintains an "anchor pool" of cached KV-cache deviations under varying prefix contexts. When agents receive overlapping content but at different prefix offsets within their respective contexts, vLLM's content-hash prefix cache cannot reuse it (the byte sequence isn't identical from token 0). KVCOMM uses approximate KV adjustment via the anchor pool to enable reuse anyway.

**Numbers:** >70% cache reuse and 7.8× speedup in 5-agent fully-connected scenarios at 1K input + 512 prefix/output tokens (TTFT ~430 ms → ~55 ms).

**Axis:** widens the *surface* of reusable KV by tolerating prefix offset variance. Sits below mechanisms that decide *what* to reuse.

**Why it's relevant for KV-cache-for-agents work:**
Orthogonal layer, not a competitor. Approaches that rely on byte-identical prefix caching could compose with KVCOMM to widen the conditions under which their selected KV is reusable. Frame in the 2×2: {static-graph, learned-prediction} × {byte-identical, approximate reuse} — KVCOMM fills the static × approximate quadrant.

Don't try to address offset-variance directly; defer to KVCOMM as a future-work composition target.
