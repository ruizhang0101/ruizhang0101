---
name: Shared 8-GPU machine — pick a free GPU, never kill others' processes
description: rtx-026 has 8 GPUs and is shared; jobs must target a free GPU and only kill PIDs they started
type: feedback
scope: host:rtx-026
---

The host `rtx-026` (172.16.176.28) has **8 GPUs** and is **shared with other users**. Hard constraints:

1. **Never kill processes you didn't start.** Pattern-based kills like `pkill -9 -f "VLLM::EngineCore"` or `pkill -9 -f "api_server"` will kill *everyone's* vLLM, not just yours. Same for iterating over `nvidia-smi --query-compute-apps=pid` and killing all of them — that wipes the entire machine.
2. **Pick a free GPU before starting.** Check `nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits` and select a GPU with low utilization. Don't assume GPU 0.
3. **Set `CUDA_VISIBLE_DEVICES=<chosen>`** in the env when launching vLLM (or anything that allocates GPU memory). Don't blast across all 8.
4. **Only kill PIDs you tracked.** Capture `$!` after backgrounding and remember it; kill that specific PID at cleanup. Use `pgrep -P <wrapper_pid>` if you need children.

**Why:** Past drivers used broad `pkill -f` patterns that would kill another user's vLLM mid-experiment. The damage isn't visible to the agent but it is to them. This rule is non-negotiable.

**How to apply:**
- Before starting any new long-running GPU job: `nvidia-smi --query-gpu=index,memory.used,memory.total --format=csv,noheader,nounits` and pick the GPU with the most headroom (default: lowest-numbered GPU with `memory.used < 1000 MB`).
- All vLLM startup invocations should set `CUDA_VISIBLE_DEVICES=$picked_gpu` explicitly.
- Replace any `pkill -f "api_server"` / `pkill -f "VLLM::EngineCore"` with PID-tracked kills:
  ```bash
  start_vllm() {
    ...
    nohup .venv/bin/python ... > $log 2>&1 &
    VLLM_PID=$!         # capture once, the actual backgrounded process
    echo $VLLM_PID
  }
  stop_vllm() {
    local pid=$1
    kill -TERM $pid 2>/dev/null
    sleep 5
    kill -KILL $pid 2>/dev/null
    # do NOT pkill -f, do NOT iterate nvidia-smi compute-apps
  }
  ```
- For checking "is my old vLLM still running?": only test your own tracked PIDs.
- If you inherit a project with broad `pkill -f` patterns in its scripts, flag them — they need retrofitting before the next campaign.

**Related:** `vllm-pid-tracked-cleanup.md` covers the cleanup recipe in more detail.
