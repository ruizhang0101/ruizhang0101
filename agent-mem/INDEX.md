# Agent Memory Index

One-line summary of every entry. Pull the file when relevant; check `scope` in
the frontmatter before applying.

## tooling/
- [python-uv](tooling/python-uv.md) — use `uv run` instead of `source .venv/bin/activate &&` for Python commands [global]
- [shared-gpu-rtx026](tooling/shared-gpu-rtx026.md) — 8-GPU shared host: pick a free GPU, only kill PIDs you started, no pattern-based pkills [host:rtx-026]
- [vllm-pid-tracked-cleanup](tooling/vllm-pid-tracked-cleanup.md) — clean up vLLM servers after experiments using tracked PIDs (not `pkill -f`) [stack:vllm]

## conventions/
- [ml-experiment-seeds](conventions/ml-experiment-seeds.md) — default 3 seeds (10, 20, 30) for new experiments unless specified [global]
- [autogen-selector-model](conventions/autogen-selector-model.md) — SelectorGroupChat needs Llama-3.1-8B-Instruct or stronger; gpt-oss-20b pingpongs [stack:autogen]
- [cs-paper-figures](conventions/cs-paper-figures.md) — vector PDF, column-width sizing, 8–10 pt fonts, ≤4 colorblind-safe colors, locked per-system colors [area:cs-systems-papers]

## references/
- [autogen-vllm-tool-wiring](references/autogen-vllm-tool-wiring.md) — three traps (model_info, vLLM tool parser, intercepted create signature) to capture tool schemas through AutoGen+vLLM [stack:autogen+vllm]
- [kv-cache-sizing](references/kv-cache-sizing.md) — per-token/per-block KV bytes formula, vLLM log signals, pressure-recipe for `--num-gpu-blocks-override` [stack:vllm]

## research/kv-cache-for-agents/
- [landscape](research/kv-cache-for-agents/INDEX.md) — 5-paper landscape of KV-cache-for-agent-serving prior art, with mechanism axes [area:kv-cache-agent-serving]
- [InfiniGen](research/kv-cache-for-agents/infinigen.md) — OSDI'24, within-iteration KV prefetch via SVD-skewed Q/K
- [KVFlow](research/kv-cache-for-agents/kvflow.md) — Jul 2025, statically-declared Agent Step Graph + step-aware eviction
- [Tokencake](research/kv-cache-for-agents/tokencake.md) — Oct 2025, multi-agent space partition + tool-call-aware offload
- [KVCOMM](research/kv-cache-for-agents/kvcomm.md) — Oct 2025, anchor pool for approximate KV reuse across non-matching prefixes
- [Continuum](research/kv-cache-for-agents/continuum.md) — Nov 2025, TTL-based KV pinning for ReAct tool-using agents
