---
name: SelectorGroupChat needs a competent selector model
description: AutoGen multi-agent claims depend on the selector following workflow rules; small instruction-tuned models pingpong 2-3 agents instead of the declared N
type: feedback
scope: stack:autogen
---

`autogen_agentchat.teams.SelectorGroupChat` routes turns by asking a selector LLM "pick the next speaker from {participants}, following these rules: ...". Whether the workflow actually exercises all declared agents depends *entirely* on the selector's instruction-following ability.

**Empirical (6 declared agents — Planner / Architect / Coder / Tester / Reviewer / Documenter):**
- `gpt-oss-20b`: pingpongs Planner ↔ Architect; Coder gets ~5% of turns; Tester / Reviewer / Documenter never called. Pattern is consistent across task sources → it's the model, not the workload.
- `Llama-3.1-8B-Instruct`: routes through all 6 with non-trivial conditional probabilities (e.g. Architect→Planner 0.60–1.00 across workloads, Coder→Reviewer 0.43–0.71).

**Why this isn't a framework bug:** AutoGen passes the right prompt; the selector just doesn't follow it. Switching to `RoundRobinGroupChat` or `GraphFlow` with hardcoded edges sidesteps the issue but defeats the point of "supervisor-routed multi-agent."

**How to apply:**
- For any multi-agent capture intended to demonstrate "richer agent topology than 2-state alternation": use `Llama-3.1-8B-Instruct` or stronger. Don't trust `gpt-oss-20b` as a `SelectorGroupChat` selector.
- This applies to *any* `SelectorGroupChat` setup, not just 6-agent. Even 4-agent configurations will likely collapse to 2-agent pingpong on weak selectors.
- Document the selector model alongside the agent count. "6-agent AutoGen workload" without specifying the selector is an incomplete description.
- If you must use a weaker selector, sanity-check the transition distribution before running the full campaign.
