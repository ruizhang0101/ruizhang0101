---
name: docstring-like-yihua
description: Write or revise Python docstrings and code comments in the LMCache house style (Google-style docstrings, why-focused comments, explicit caller contracts and thread-safety notes), as practiced by Yihua Cheng (ApostaC). Use when authoring new Python code, adding/fixing docstrings, or making comments read like the LMCache codebase.
allowed-tools: Read, Edit, Write, Grep, Glob
argument-hint: "[file path or symbol to document]"
---

# Docstring & Comment Style (LMCache / ApostaC house style)

Write Python docstrings and inline comments the way Yihua Cheng (`ApostaC`) writes
them in LMCache. This style was reverse-engineered from his merged PRs and matches
the project's own `docs/coding_standards.md` Section 3.

The one-sentence summary of the style: **Google-style docstrings that document the
contract (args, returns, raises, thread-safety, caller obligations), and sparse
inline comments that explain *why*, never *what*.**

## When to use this

Invoke when the task is to:
- Add docstrings/comments to new Python code you just wrote.
- Fix or upgrade existing docstrings (e.g., during a pre-PR pass).
- Make code "read like the rest of the LMCache codebase."

`$ARGUMENTS` may name a file or a symbol. If given, focus on that target; otherwise
apply to the code under discussion. Do not rewrite logic — only documentation and
comments, unless explicitly asked.

---

## 1. Core principles (non-negotiable)

1. **Google-style docstrings.** Sections are `Args:`, `Returns:`, `Raises:`,
   `Note:`, and (for dataclasses) `Attributes:`. No NumPy/reST section blocks.
2. **Document the contract, not the implementation.** A docstring describes what a
   caller must know to use the function correctly: argument constraints, what the
   return value means, what is raised, thread-safety, and ordering obligations.
3. **Describe actual current behavior.** Never document planned/future behavior. If
   a parameter is accepted but currently a no-op, the docstring must say so.
4. **Comments explain *why*, code shows *what*.** Do not narrate obvious lines. Add
   a comment only for intent, a subtle invariant, a gotcha, a concurrency hazard, or
   a performance trade-off.
5. **Make caller obligations and thread-safety explicit.** This is the signature
   trait of the style: "Thread-safe.", "Caller must hold `self._lock`.", "call X
   before Y or the buffer leaks."

---

## 2. Docstring format rules

- **Triple-quote placement:** opening `"""` is on the **same line** as the summary.
  Closing `"""` is on its own line for multi-line docstrings; single-line docstrings
  are `"""Summary on one line."""`.
- **Summary line:** one sentence, **imperative mood** for functions/methods
  ("Register…", "Return…", "Query…", "Resolve…"), ends with a period. For classes,
  a noun phrase is fine ("Thread-safe registry mapping…"). Properties may use a
  short noun phrase ("Path of the trace file on disk.") or imperative ("Return the
  file header.").
- **Blank line** after the summary, before any detail paragraph or section header.
  No blank line between a section header (`Args:`) and its first entry.
- **Args entries:** `name: Description.` — **a colon, no type** (types live in the
  signature), description starts capitalized, ends with a period. Wrap long
  descriptions with a 4-space hanging indent under the arg name.
- **Returns / Raises:** describe the *meaning* of the value or the *condition* that
  triggers each exception, not just the type.
- **Code references:** use double backticks ``` ``like_this`` ``` in docstrings (the
  docs render as reStructuredText). Sphinx cross-reference roles are welcome in
  module/class docstrings: `` :func:`...` ``, `` :meth:`...` ``, `` :class:`...` ``,
  `` :data:`...` ``, `` :mod:`...` ``.
- **Wrap** prose to ~79 columns.

### When a docstring is required (from `docs/coding_standards.md` §3.1)

| Scope | Requirement |
|-------|-------------|
| Public functions and methods | Full docstring (always) |
| Module-level / global helpers | Full docstring |
| Long private helpers | Full docstring |
| Short, clear class-private helpers | Short one-liner acceptable |
| Override methods with no new behavior | Short one-liner acceptable |

A **full** docstring has, in order: summary → optional detail paragraph(s) →
`Args:` → `Returns:` → `Raises:` → `Note:`. Omit a section only when it genuinely
has no content (e.g., no parameters, returns `None`, raises nothing).

### Templates

Method / function (full):

```python
def query_lookup_result(self, request_id: PrefetchRequestId) -> int | None:
    """Query the number of prefix hits from the lookup phase.

    Thread-safe. Returns the number of prefix hits if the lookup phase
    has completed, None if still in progress, or if the prefetch request
    has already been consumed by ``query_prefetch_result``.

    Args:
        request_id: The request ID from ``submit_prefetch_request``.

    Returns:
        Number of prefix hits from the lookup phase, or None if not yet
        complete or if the request was already consumed.

    Note:
        This function does not pop the result. The caller must call
        ``query_prefetch_result`` afterward, otherwise nobody cleans up
        the completed-lookups dictionary, causing a memory leak.
    """
```

One-liner (simple accessor / obvious helper):

```python
@staticmethod
def get() -> PeriodicEventNotifier | None:
    """Return the singleton instance, or None if not created."""
```

Class:

```python
class ClientPollingLoop:
    """Singleton polling loop shared by all MessageQueueClient instances.

    Instead of each client running its own daemon thread and zmq.Poller,
    a single loop polls all clients' DEALER sockets and dispatches
    inbound/outbound work.

    Use ``get_instance()`` / ``release_instance()`` for lifecycle
    management — the loop starts lazily on the first client and stops
    automatically when the last client releases.
    """
```

Dataclass (use `Attributes:`, or per-field docstrings for invariants):

```python
@dataclass
class HandlerSpec:
    """Specification for a single message queue handler.

    Args:
        request_type: The ZMQ request type this handler serves.
        handler: The callable that processes the request.
        pool: Which thread pool the handler runs in.
    """

    prefetch_request_id: int
    """Opaque ID for tracking L2 prefetch in the controller.
    -1 if no L2 request was submitted."""
```

Module (narrative; explain the *why* and the public surface):

```python
"""``lmcache trace`` — inspect and replay storage-level trace files.

Subcommands:

* ``info FILE`` — print a summary (header metadata + per-qualname counts).
* ``replay FILE ...`` — reissue every recorded call against a fresh
  StorageManager, honoring the recorded inter-call timings.

Trace *capture* is not a ``trace`` subcommand — recording is bound to the
live process via ``lmcache server --trace-level storage``.
"""
```

---

## 3. Comment style rules

- **Why, not what.** If the comment restates the code, delete it.
- **Voice:** capitalized, full sentences with periods for explanations; lowercase
  fragment with no period for short labels (`# Pending job's futures`,
  `# Outbound:`). Use double backticks for code names and em-dashes (`—`) for asides.
- **`# NOTE:`** for an invariant or gotcha a future editor would otherwise miss.
  **`# TODO:`** for deferred work (optionally `# TODO(name):`).
- **Section-label comments** to mark phases of a routine: `# Inbound: dispatch each
  ready DEALER socket to its client.`, `# Step 8: update the lookup result …`.
- **Data-structure map comments** above container fields, stating the mapping and
  any sentinel:

  ```python
  # Key: (model_name, world_size) -> MemoryLayoutDesc
  self._registry: dict[tuple[str, int], MemoryLayoutDesc] = {}

  # poly_chunk_hash -> compact_chunk_id; -1 = empty
  self._table_id = np.full(self._TABLE_SIZE, -1, dtype=np.int64)
  ```

- **Constant annotations** giving the magnitude or origin:

  ```python
  _TABLE_BITS: int = 20  # 2^20 ~ 1 M entries
  _BASE: np.uint64 = np.uint64(0x9E3779B97F4A7C15)  # Fibonacci-hashing constant
  ```

- **Concurrency hazards** spelled out explicitly:

  ```python
  # Skip tokens that overlap with APC-cached blocks to avoid a data race:
  # the retrieve writes on the LMCache CUDA stream while concurrent requests
  # may read those same APC-shared blocks on the vLLM CUDA stream.
  ```

- **Performance trade-offs** with the asymptotics:

  ```python
  # Sort once per summary call — ``record`` keeps the list unsorted (O(1)
  # append) so total work is O(N log N) per summary, not O(N) per insert.
  ```

- **Design-justification blocks** for non-obvious choices — explain why the simpler
  approach was rejected, and cite the PR/bug when relevant ("see bugbot comment on
  PR #3075"). Keep them tight; no blank `#` padding lines unless separating
  paragraphs.

---

## 4. Anti-patterns to avoid

- ❌ Type info duplicated in the docstring (`request_id (int): …`). Types are in the
  signature; write `request_id: …`.
- ❌ NumPy/reST headings (`Parameters\n----------`). Use Google `Args:`.
- ❌ Comments that restate code (`# increment counter` above `counter += 1`).
- ❌ Documenting future/aspirational behavior, or omitting that a param is a no-op.
- ❌ Praise/filler prose, decorative ASCII banners (`# ===== SECTION =====`) inside
  functions — prefer a one-line label comment.
- ❌ `Optional[X]` in signatures (use `X | None`) and bare generics (`dict` → `dict[K, V]`);
  these are style violations the docstrings sit alongside, so keep them consistent.

---

## 5. Application checklist

When adding/revising docstrings and comments:

- [ ] Every public function/method/class has a full docstring; trivial private
      helpers/overrides may have a one-liner.
- [ ] Summary is one imperative sentence (noun phrase for classes), ends with a period.
- [ ] `Args:` lists every parameter as `name: Description.` with constraints noted
      (e.g., "length of `keys` must equal length of `values`").
- [ ] `Returns:` explains what the value *means*; ambiguous returns are disambiguated.
- [ ] `Raises:` lists each exception with its trigger condition.
- [ ] Thread-safety stated when relevant ("Thread-safe.", "Caller must hold the lock.").
- [ ] Caller ordering/cleanup obligations stated in a `Note:` when they exist.
- [ ] Docstring matches actual current behavior; no-op params flagged as such.
- [ ] Code references use double backticks; module/class docstrings may use Sphinx roles.
- [ ] Comments explain *why*; data-structure fields have map/sentinel comments; no
      restate-the-code comments.

## More verbatim examples

See `references/examples.md` for a larger gallery of real docstrings and comments
quoted verbatim from his PRs, grouped by category, to pattern-match against.
