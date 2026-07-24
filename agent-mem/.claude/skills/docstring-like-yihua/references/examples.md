# Verbatim style gallery (ApostaC / LMCache)

Real docstrings and comments quoted verbatim from Yihua Cheng's merged LMCache PRs
(#3423, #3409, #2818, #3391, #3063, #2889, #3075, #2838). Use these as gold-standard
patterns to imitate. Nothing here is invented.

---

## Docstrings

### Class docstrings

```python
class ClientPollingLoop:
    """Singleton polling loop shared by all MessageQueueClient instances.

    Instead of each client running its own daemon thread and zmq.Poller,
    a single loop polls all clients' DEALER sockets and dispatches
    inbound/outbound work.

    Use ``get_instance()`` / ``release_instance()`` for lifecycle
    management — the loop starts lazily on first client and stops
    automatically when the last client releases.
    """
```

```python
class LayoutDescRegistry:
    """Thread-safe registry mapping (model_name, world_size) to MemoryLayoutDesc.

    Modules write to this registry when KV caches are registered.
    Consumers (e.g. LookupModule) read from it to find layout descriptors
    for prefetch tasks.
    """
```

```python
class MPCacheEngineContext:
    """Shared infrastructure for all engine modules.

    Holds the storage manager, token hasher, session manager, event bus,
    and layout descriptor registry. Modules receive this context at init
    and use it for shared operations.

    Args:
        storage_manager_config: Configuration for the storage manager.
        chunk_size: Chunk size for KV cache operations.
        hash_algorithm: Hash algorithm for token hashing.
    """
```

### Method docstrings — Args / Returns / Raises / Note

```python
def query_lookup_result(self, request_id: PrefetchRequestId) -> int | None:
    """
    Query the number of prefix hits from the lookup phase.

    Thread-safe. Returns the number of prefix hits if the lookup phase
    has completed, None if still in progress, or the prefetch request
    has already been consumed by query_prefetch_result.

    Args:
        request_id: The request ID from submit_prefetch_request.

    Returns:
        Number of prefix hits from the lookup phase, or None if not yet complete
        or if the request has already been consumed by a previous call to this
        method.

    Note:
        This function does not pop the result. The caller need to make sure to call
        the query_prefetch_result after calling this function, otherwise nobody
        will clean up the completed lookups dictionary, causing memory leak.
    """
```

```python
def resolve_obj_keys(self, key: IPCCacheEngineKey) -> list[ObjectKey]:
    """Resolve object keys from an IPC cache key.

    Uses the session manager to track token state and the token hasher
    to compute chunk hashes for the requested range.

    Args:
        key: IPC cache key describing model/session/token range.

    Returns:
        Resolved object keys for the requested token range.

    Raises:
        ValueError: If ``key.worker_id`` is ``None``.
    """
```

```python
def cb_store_pre_computed(
    self,
    key: IPCCacheEngineKey,
    offset: int,
    instance_id: int,
    event_ipc_handle: bytes,
) -> tuple[bytes, bool]:
    """Store the pre-computed chunks in the underlying storage for later retrieval.

    Args:
        key: IPCCacheEngineKey containing the token ids for which the
            pre-computed chunks are stored.
        offset: The starting offset in the CB KV cache buffer where the
            pre-computed chunks begin.
        instance_id: The instance_id of the blend engine instance to store
            the pre-computed chunks for.
        event_ipc_handle: The IPC handle for the CUDA event that signals the
            completion of LLM inference.

    Returns:
        IPC handle bytes for the event that signals the completion of storing
        the pre-computed chunks, and a boolean flag indicating if the store
        is successful.

    Raises:
        ValueError: If instance_id is not registered for CB KV cache.

    Note:
        The input tokens should not have any separator in it. It should just
        be one "paragraph".
        This function will discard the last partial chunk and only store the
        full chunks.
    """
```

```python
def register(self, qualname: str, handler: Handler) -> None:
    """Register a handler for *qualname*.

    Args:
        qualname: Fully-qualified call-site name exactly matching
            the ``qualname`` field written by the recorder.
        handler: Callable invoked on each matching record.

    Raises:
        ValueError: If a handler is already registered for
            *qualname*.
    """
```

```python
@staticmethod
def create(interval_ms: int, use_eventfd: bool) -> None:
    """Create the singleton. Idempotent -- second call is a no-op.

    Args:
        interval_ms: Notification interval in milliseconds.
        use_eventfd: True to write as eventfd (8-byte uint64),
            False to write as pipe (1-byte).
    """
```

```python
def attach_storage_config(self, config: StorageManagerConfig) -> None:
    """Write the header populated from the StorageManagerConfig.

    Must be called before any records are written; the
    server lifecycle does this immediately after construction.
    Subsequent calls are silently ignored — the header is written
    once for the lifetime of the file.

    Args:
        config: The StorageManagerConfig in use.  Its dataclass
            form is JSON-serialized and SHA-256 hashed for the
            header digest, so a replay driver can detect
            mismatched configurations.
    """
```

### One-liner docstrings

```python
@staticmethod
def get() -> PeriodicEventNotifier | None:
    """Return the singleton instance, or None if not created."""
```

```python
@property
def chunk_size(self) -> int:
    """Chunk size for KV cache operations."""
    return self._chunk_size
```

```python
@property
def num_heads(self) -> int:
    """Returns the number of attention heads in the model"""
    return self.num_heads_
```

### Abstract-method docstrings (document the contract for implementers)

```python
@abstractmethod
def notify_fileno(self) -> int:
    """Return the fd to write to for signaling.

    For eventfd this is the same as ``fileno()``.  For pipes
    this is the *write* end (``fileno()`` returns the read end).
    """
```

### Module docstrings (narrative, why-focused)

```python
"""``lmcache trace`` — inspect and replay storage-level trace files.

Subcommands:

* ``info FILE`` — print a summary (header metadata + per-qualname
  record counts).
* ``replay FILE ...`` — reissue every recorded call against a fresh
  StorageManager, honoring the recorded inter-call timings. ...

Trace *capture* is not a ``trace`` subcommand — recording is bound to
the live process via ``lmcache server --trace-level storage
[--trace-output ...]``. ...
"""
```

```python
"""Protocol and types for pluggable engine modules."""
```

### Dataclass field docstrings (invariant inline)

```python
prefetch_request_id: int
"""Opaque ID for tracking L2 prefetch in the controller.
-1 if no L2 request was submitted."""
```

### Test docstrings (imperative "Test that …")

```python
def test_shared_loop_lifecycle():
    """
    Test that multiple clients share a single ClientPollingLoop and
    that the loop is torn down when all clients close.
    """
```

---

## Inline / block comments

### Data-structure map + sentinel comments

```python
# Key: (model_name, world_size) -> MemoryLayoutDesc
self._registry: dict[tuple[str, int], MemoryLayoutDesc] = {}

# poly_chunk_hash -> compact_chunk_id; -1 = empty
self._table_id = np.full(self._TABLE_SIZE, -1, dtype=np.int64)

# compact_chunk_id -> caller-supplied token_hash (full bytes)
self._chunk_token_hash: list[bytes | None] = []

# Thread-safe lookup results (background -> external)
self._lookup_results_lock = threading.Lock()
```

### Constant annotations

```python
_TABLE_BITS: int = 20  # 2^20 ~ 1 M entries
_BASE: np.uint64 = np.uint64(0x9E3779B97F4A7C15)  # Fibonacci-hashing constant
```

### Section-label / step comments

```python
# Inbound: dispatch each ready DEALER socket to its client.
for sock, event in socks.items():

# Outbound: shared notifier woke us — drain it, then flush
# all clients' output queues.

## Step 8: update the lookup result based on the final load plan
self._update_lookup_results(request.request_id, prefix_length)

# Drain remaining ops so any waiting threads unblock.
self._process_ops()
```

### `# NOTE:` invariants

```python
# NOTE: make sure we only edit the pending_futures dict in this thread
```

```python
# NOTE: Store is not batched because some obj_keys may be
# skipped (not in reserved_dict), making block_ids
# non-contiguous. Batching would require torch.cat to
# reassemble block_ids, negating the benefit.
```

### `# TODO:`

```python
# TODO: implement get_gpu_buffer_batched
tmp_buffers = gpu_context.get_tmp_gpu_buffer_batched(
```

### Concurrency hazards

```python
# Skip tokens that overlap with APC-cached blocks to
# avoid a data race: the retrieve writes on the LMCache
# CUDA stream while concurrent requests may read from
# those same APC-shared blocks on the vLLM CUDA stream.
```

### Performance trade-off reasoning

```python
# Sort once per summary call — ``record`` keeps the
# list unsorted (O(1) append) so the total work is
# O(N log N) per summary rather than O(N) per insert.
```

```python
# Single write so the prefix and body land atomically for frames
# below PIPE_BUF; two writes would let a concurrent appender
# interleave between them and would also double the syscall
# count on an unbuffered fd.  Caller holds ``self._lock`` (or is
# in __init__ before the gate flips on).
```

### Design-justification blocks (why the simpler way was rejected; cite PRs/bugs)

```python
# Each resource is acquired under its own try/except so that a
# failure partway through __init__ still releases what has
# already been opened.  Without this, ``__exit__`` / ``close``
# never runs (the caller never got a valid instance), and the
# reader / bus would leak — see Cursor bugbot comment on
# PR #3075.
```

```python
# ``required=True`` on the subparser makes this unreachable
# in practice; branch is kept for defensive logging.
```

```python
# Manual TRACE_CALL emission for the context manager.  The
# ``@enable_tracing`` decorator cannot wrap a ``@contextmanager``
# generator function (it would publish the call to the wrapper
# rather than to ``__enter__``).  Emit enter/exit events
# directly, gated on the tracing flag for zero overhead when
# disabled.
```

---

## Style fingerprints (quick reference)

- Google-style sections only: `Args:` `Returns:` `Raises:` `Note:` `Attributes:`.
- Summary on the same line as opening `"""`; imperative verb; ends with a period.
- `Args:` entries are `name: Description.` — colon, **no type**, period.
- Double backticks ``` ``code`` ``` in docstrings; Sphinx roles (`:func:` `:meth:`
  `:class:` `:data:` `:mod:`) in module/class docstrings.
- Em-dash (`—`) for asides; `--` sometimes used as a sentence separator.
- Heavy on **thread-safety** ("Thread-safe.", "Caller holds `self._lock`") and
  **caller obligations** ("call X before Y or it leaks").
- Comments are *why*-focused; map comments (`# a -> b; -1 = empty`) on container
  fields; performance and concurrency rationale spelled out; cites PR numbers.
- No emoji, no decorative banners inside functions.
