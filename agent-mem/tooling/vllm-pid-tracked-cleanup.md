---
name: Clean up vLLM servers after experiments (PID-tracked, not pattern-based)
description: Always kill vLLM API servers and EngineCores when an experiment finishes — using tracked PIDs, never `pkill -f`
type: feedback
scope: stack:vllm
---

After running any experiment that starts `vllm.entrypoints.openai.api_server` (or a wrapper like `benchmarks/server.py`), **kill the server processes you started** before reporting results. Don't leave zombie `VLLM::EngineCore` processes holding GPU memory.

**Why:** Stale engine cores accumulate and consume GPU memory, blocking the next run. On a shared host they also degrade other users' jobs. rui has been bitten by "17 GB free on a 95 GB GPU" because of prior runs' zombies, forcing a manual cleanup sweep before the new server can start.

**How to apply:**

On a **single-user / dedicated** host:
```bash
ps auxww | grep api_server | grep -v grep | awk '{print $2}' | xargs -r kill -9
sleep 2
ps auxww | grep "VLLM::EngineCore" | grep -v grep | awk '{print $2}' | xargs -r kill -9
```

On a **shared** host (e.g. `rtx-026` — see `shared-gpu-rtx026.md`): the pattern-based version above is **forbidden** because it kills other users' vLLMs. Use PID-tracked cleanup instead:

```bash
# at startup
nohup .venv/bin/python -m vllm.entrypoints.openai.api_server ... > $log 2>&1 &
VLLM_PID=$!

# at teardown
kill -TERM $VLLM_PID 2>/dev/null
sleep 5
kill -KILL $VLLM_PID 2>/dev/null
# children (EngineCore workers) usually exit with the parent;
# if not, `pgrep -P $VLLM_PID` to find and kill them by PID
```

Verify GPU memory frees: `nvidia-smi --query-gpu=memory.free --format=csv,noheader`. Only consider the experiment complete after cleanup.

Applies to any vLLM server the agent spawned — benchmark, audit, probe, or ad-hoc. Does **not** apply to vLLM servers rui started manually unless they ask.
