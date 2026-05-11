---
name: KV cache for LLM agent serving — paper landscape
description: Five papers framing the prior art on KV-cache management for multi-agent / tool-using LLM serving, positioned on mechanism axes
type: research
scope: area:kv-cache-agent-serving
---

Five papers form the prior-art landscape for KV-cache management in
multi-agent / tool-using LLM serving as of late 2025.

| Paper | Date | Axis | Mechanism |
|---|---|---|---|
| [InfiniGen](infinigen.md) | OSDI'24 | layer-to-layer within one inference step | SVD-skewed Q/K + speculate layer i from layer i−1 attention input |
| [KVFlow](kvflow.md) | Jul 2025 | within-workflow (statically declared) | Agent Step Graph, steps-to-execution priority |
| [Tokencake](tokencake.md) | Oct 2025 | within-program tool-call idle time | space partitioning + time scheduler |
| [KVCOMM](kvcomm.md) | Oct 2025 | offset-variance reuse layer | anchor pool of KV deviations |
| [Continuum](continuum.md) | Nov 2025 | within-program tool-call retention | TTL-based KV pinning + cost-benefit model |

**Two-by-two design space:**

|                              | byte-identical reuse | approximate reuse |
|------------------------------|----------------------|-------------------|
| **statically-declared graph** | KVFlow               | (open)            |
| **learned prediction**        | (e.g. CacheSage)     | future work, possibly KVCOMM-composed |

**Workload taxonomy — which paper wins where:**
- Within-session multi-turn long-context → **InfiniGen**
- ReAct tool-using single-program (SWE-Bench) → **Continuum**
- Statically-declared multi-agent workflow → **KVFlow**
- Multi-tenant routing / persona / shared-prompt agentic serving with dynamic LLM-selected routing → open (CacheSage-style)

**Methodological bar set by the field (especially Continuum):**
≥2 hardware × ≥2 models × ≥3 baselines × P90/P95 latency × ablation × turn-count scaling × open-source code.

**Notes on these summaries:**
Originally written for the CacheSage paper. Technical content (mechanism descriptions, numbers, evaluation setup) is reusable; the "Why this matters for CacheSage" sections are project-specific framing — useful only if a future project is in the same space.
