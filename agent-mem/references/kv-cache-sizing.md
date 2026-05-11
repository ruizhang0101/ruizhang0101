---
name: KV cache sizing — formulas + how to re-derive for any model + pressure recipe
description: Per-token / per-block KV bytes, GPU budget math, vLLM log signals, and recipe for choosing `--num-gpu-blocks-override` at a target cache-pressure ratio
type: reference
scope: stack:vllm
---

## General formula (model-agnostic)

```
per_token_kv_bytes  = 2 × num_layers × num_kv_heads × head_dim × dtype_bytes
per_block_bytes     = block_size_tokens × per_token_kv_bytes      (vLLM block_size = 16 by default)
auto_num_gpu_blocks = floor(available_kv_memory_bytes / per_block_bytes)
```

Where `dtype_bytes = 2` for bf16/fp16, `1` for int8, `0.5` for int4/mxfp4. The `2` outside is K and V; do **not** double-count when including kv_cache dtype.

## Authoritative source — re-derive for any new model

**Don't trust calculations alone. Verify by reading the vLLM startup log.** The relevant lines:

```
INFO ... [gpu_worker.py:436] Available KV cache memory: <X> GiB
INFO ... [kv_cache_utils.py:833] Overriding num_gpu_blocks=<auto_value> with num_gpu_blocks_override=<override>
INFO ... [kv_cache_utils.py:1323] GPU KV cache size: <T> tokens
INFO ... [kv_cache_utils.py:1328] Maximum concurrency for <max_model_len> tokens per request: <ratio>x
```

From the auto value, derive: `per_block_bytes = available_kv_memory / auto_num_gpu_blocks`. Confirms the formula. Recipe to extract:

```
grep -E "Available KV cache|num_gpu_blocks|GPU KV cache size|Maximum concurrency" /tmp/<vllm_log>.log
```

## Worked example — gpt-oss-20b on A6000, gpu-mem=0.5, mxfp4

```
Architecture          : 24 layers, 8 KV heads, 64 head_dim, 2 KV cache groups, kv_cache_dtype=bf16
per_token_kv_bytes    = 2 × 24 × 8 × 64 × 2 = 49,152 bytes ≈ 48 KB
per_block_bytes (16t) = 16 × 48 KB          = 768 KB
Available KV memory   = 30.12 GiB           (after 10 GB mxfp4 weights, ~2 GB activations/cudagraphs)
auto num_gpu_blocks   = 41,127              (vLLM-computed without override)
auto cache size       = 657,952 tokens      (~40× one 16K-context request)
Model weights size    ≈ 10 GB               (20B params × 0.5 bytes mxfp4)
```

## Experimental pressure recipe

Working backwards from a target pressure regime:

```
1. Estimate workload working set W in tokens
   Example: AutoGen capture = 3 agents × 200-tok system + 4 concurrent × 3K context ≈ 13K tokens
2. Pick pressure ratio P
   tight: 0.3 – 0.6   (forces eviction)
   moderate: 1.0 – 2.0
   loose: 3.0 – 5.0   (no eviction pressure)
3. Compute:
   target_tokens = W × P
   target_blocks = target_tokens / vLLM_block_size  (16 default)
4. Verify GPU budget: required_KV_bytes = target_blocks × per_block_bytes
   ensure (model_weights + activations + required_KV_bytes) ≤ (GPU_total × gpu_mem_util)
5. Set --num-gpu-blocks-override target_blocks
```

Worked example (W ≈ 13K tokens):
- Tight (P=0.4) → 5,200 tokens → 325 blocks (use 300 for headroom)
- Loose (P=4)   → 52K tokens   → 3,250 blocks

## Pressure level reference (gpt-oss-20b, the setup above)

| Override | Tokens | vs auto | Concurrency at 16K-context |
|---|---:|---:|---:|
| 200 | 3,200 | 0.5% | 0.20× |
| 300 | 4,800 | 0.7% | 0.29× |
| 500 | 8,000 | 1.2% | 0.49× |
| 1,000 | 16,000 | 2.4% | 0.98× |
| 2,000 | 32,000 | 4.9% | 1.95× |
| auto (~41K) | 657K | 100% | 40.2× |

## Caveats

- `Hybrid KV cache manager` is disabled when `--kv-transfer-config` (CPU offload) is set. This means sliding-window-attention models lose memory savings; for non-SWA models the impact is small but worth checking on new models.
- The vLLM log `kv_cache_dtype=auto` resolves to bf16 by default. If `kv_cache_dtype=fp8` is set explicitly, halve the per-token bytes.
- `chunked_prefill` and `max_num_batched_tokens` affect prefill batching but not KV cache sizing.
- gpt-oss-20b uses 2 KV cache groups internally (visible in `BlockHashWithGroupId`). The per-token bytes already account for total KV across groups.

## Action when changing model

1. Look up `num_hidden_layers`, `num_key_value_heads`, `head_dim` from the model's `config.json`.
2. Run a smoke vLLM startup with target `--gpu-mem` and read the log.
3. Confirm `Available KV cache memory` and `auto num_gpu_blocks` match the formula.
4. Recompute the pressure table above with the new per_block_bytes.
