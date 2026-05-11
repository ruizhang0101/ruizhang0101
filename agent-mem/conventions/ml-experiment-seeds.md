---
name: Default 3 seeds per experiment unless specified
description: Default to 3 seeds (10, 20, 30) for new experiments, ablations, or sweeps; extend on request
type: feedback
scope: global
---

Default to **3 seeds** for any new experiment cell, ablation, or sweep unless rui specifies otherwise.

**Why:** Stated policy: "from now on, if I didn't specify, all the exp runs 3 seeds." Typical experiment cells take ~10 minutes each; 3 seeds × N cells fits a sensible iteration cadence, and 5 seeds doubles wall-time without proportional information gain at the variance levels rui's experiments tend to show.

**How to apply:**
- New ablation drivers default to seeds `[10, 20, 30]`.
- "Rerun with more seeds" / "settle the variance" → extend to 5 or more.
- When citing prior results at a different seed count, preserve the original sample — don't re-run at 3 just to standardize, unless asked.
- For paper submission, explicitly note when cells are at n=3 vs n=5; reviewers will ask.
