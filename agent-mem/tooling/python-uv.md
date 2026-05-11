---
name: Use uv run instead of source activate
description: Prefer `uv run` over sourcing .venv/bin/activate for Python commands
type: feedback
scope: global
---

Use `uv run` instead of `source .venv/bin/activate &&` for running Python commands.

**Why:** Shorter, no need to activate the venv every time. Stated preference.

**How to apply:**
- `source .venv/bin/activate && python foo.py` → `uv run python foo.py`
- `source .venv/bin/activate && python -m pytest` → `uv run pytest`
- Applies to any project of rui's that uses `uv` for env management (most do).
- If a project doesn't have a `uv.lock` or `pyproject.toml`, fall back to the project's documented approach — don't force uv where it isn't set up.
