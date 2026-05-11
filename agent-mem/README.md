# Agent Memory Library

Reusable memory distilled from past projects. When an agent starts work on a new
project for rui (rzhang@tensormesh.ai), it should read `INDEX.md` first to see
what applies, then pull individual files as needed.

This is **cross-project** memory — things worth carrying forward. It is *not* a
substitute for a project's own `memory/` directory, which holds the active
project's state, decisions, and ephemera.

## Layout

- **`INDEX.md`** — one-line summary of every entry, grouped by folder. Read first.
- **`tooling/`** — environment, CLI, and host preferences. Some are machine-specific.
- **`conventions/`** — how rui wants experiments, papers, and code organized.
- **`references/`** — technical how-tos and lookup tables (formulas, wiring guides).
- **`research/`** — subject-matter notes (currently: KV cache for LLM agent serving).

## File format

Every entry has YAML frontmatter:

```
---
name: <short title>
description: <one line — used to decide relevance from INDEX alone>
type: feedback | reference | research
scope: global | host:<hostname> | area:<research-area> | stack:<tool>
---
```

`scope` is the most important field for an incoming agent. Examples:
- `global` — apply on any project for this user
- `host:rtx-026` — only on the shared 8-GPU machine
- `stack:vllm` — only when working with vLLM
- `area:kv-cache-agent-serving` — only relevant to that research area

## When to pull an entry

- The user is starting a task whose stack/host/area matches an entry's `scope`.
- A decision you're about to make is governed by an entry (default seeds, figure rules, etc.).
- The user references prior work and the topic matches a `research/` note.

## When to add to this directory

Promote a memory here when it would help in a **future, different project** —
not just the current one. Keep out:
- Dated state snapshots ("project status as of YYYY-MM-DD")
- Per-file bug logs or repo-specific fix recipes
- Paper-specific framings (motivation→design mapping, ablation cell labels)
- Anything trivially re-derivable from `git log` or reading the code

The bar: *would I want a fresh agent on a brand-new project to know this?*
If the answer is "only if they're still doing X" — fine, save it with a narrow
`scope` so it filters out for unrelated work.

## When to retire an entry

- The scope no longer exists (machine retired, research area abandoned).
- A newer entry supersedes it (note the supersession in the new file).
- Verification shows it's wrong now (file moved, library changed API, etc.).

Prefer editing over deleting when the underlying advice is still partially valid
but needs an update — preserves the reasoning trail.
