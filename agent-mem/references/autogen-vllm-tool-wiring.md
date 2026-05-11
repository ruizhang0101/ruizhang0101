---
name: AutoGen + vLLM tool-call wiring (3 traps)
description: How to make AutoGen-agentchat agents emit tool calls that flow through vLLM and back, including the model_info, tool-parser, and intercepted-create() gotchas
type: reference
scope: stack:autogen+vllm
---

When wiring AutoGen-agentchat agents through a vLLM-served model — especially when intercepting calls for trace capture — three things must align or you get either "model does not support function calling" or silent loss of tool schemas.

### 1. AutoGen client `model_info["function_calling"]`

`OpenAIChatCompletionClient(model=..., model_info=...)` defaults to `function_calling=False`. Setting `function_calling=True` is required when any agent has tools — otherwise AutoGen rejects the agent before any LLM call. The `model_info` dict must have all keys present:

```python
model_info={"vision": False, "function_calling": True,
            "json_output": False, "family": "unknown",
            "structured_output": False}
```

### 2. vLLM tool-call parser flags

For Llama-3.1 family models, vLLM needs both:

```bash
--tool-call-parser llama3_json --enable-auto-tool-choice
```

Without these, vLLM returns the model's tool call as raw text in `msg.content`, AutoGen doesn't see `msg.tool_calls`, and tool-using agents silently fail. The parser is model-family specific (`llama3_json`, `mistral`, `hermes`, etc.). Notably, `gpt-oss-20b`'s harmony format isn't supported by any built-in parser as of vLLM 0.7.

### 3. Intercepted `create()` must accept `tools=` explicitly

AutoGen's `OpenAIChatCompletionClient.create(messages, *, tools=(), tool_choice=..., ...)` passes `tools` as a keyword-only arg, **not** in `**kwargs`. A logging wrapper that does `async def create(self, messages, **kwargs)` will not see `tools` unless explicitly enumerated:

```python
async def create(self, messages, *, tools=(), **kwargs):
    # serialize tools into trace here
    return await super().create(messages, tools=tools, **kwargs)
```

### Verification

After all three fixes, captured turns should contain a `tools` field with JSON-encoded schemas. Confirm with:

```python
import pyarrow.parquet as pq
t = pq.read_table('autogen_trace.parquet').to_pandas()
assert 'tools' in t.columns
assert any(len(row['tools']) > 0 for _, row in t.iterrows())
```

Run the verification on the first 2-3 captured sessions; don't wait until a full N-task run finishes — the failure mode is silent.

### Why this matters

Tool schemas can be the largest single contribution to per-turn prompt size on tool-using workloads. A trace that silently drops them understates KV reuse opportunity, agent context size, and any downstream metric that depends on the full prompt surface.
