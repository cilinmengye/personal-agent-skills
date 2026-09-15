---
name: code-comments
description: >
  Enforce documentation and visual structure standards for systems-level
  and LLM-inference codebases (Python, C++, CUDA). Apply this skill
  whenever writing, editing, refactoring, or reviewing any .py, .cpp,
  or .cu file — including small edits. Consult before producing any code.
license: MIT
---

# Code Commenting Guidelines

Two things are enforced equally: **semantic annotation** and **visual breathing room**.
The goal is always to explain *why*, not *what* — the code already says what it does.

---

## 1. Semantic Tokens (highest priority)

This is the most important rule. Replace all informal markers with standardized tokens.
These tokens are the primary mechanism for communicating intent that cannot be inferred
from the code itself.

| Token      | When to use                                                          |
|------------|----------------------------------------------------------------------|
| `WARNING:` | Thread races, memory corruption, mandatory ordering constraints.     |
| `PERF:`    | Warp divergence, cache pressure, vectorization opportunities.        |
| `NOTE:`    | Non-obvious design rationale, mathematical invariants, API quirks.   |
| `TODO:`    | Known gaps or deferred work.                                         |
| `FIXME:`   | Known bugs pending a fix.                                            |

**Never use:** `!!!`, `***`, `IMPORTANT`, `CRITICAL`, `HACK`, or any emoji.

```python
# WRONG
# !!! sync threads before reduction !!!
__syncthreads()

# CORRECT
# WARNING: barrier is mandatory before the warp-reduction phase;
#          removing it causes non-deterministic partial sums.
__syncthreads()
```

```python
# WRONG
# NOTE: this is important
block_size = 256

# CORRECT — only write NOTE when the rationale is genuinely non-obvious
# NOTE: 256 threads saturates a single SM on Ampere without spilling
#       registers; larger values cause occupancy regression.
block_size = 256
```

---

## 2. Breathing Room

Code that looks "pasted together" is forbidden. Logical phases must be visually separated
and labelled. Comments inside function bodies explain the *intent* of each phase, not the
mechanics of each line.

**Spacing rules (concrete numbers, no exceptions):**

- **1 blank line** whenever the semantic responsibility of the code changes —
  even within a single logical phase. Setup code and computation code are
  different responsibilities; separate them.
- **1 blank line** between distinct logical phases inside a function body.
- **2 blank lines** between methods or top-level functions.
- **3 blank lines** above a zone anchor (see §4).
- Long argument lists break across lines with a trailing comma.
- Every `if`/`else` branch whose body spans more than one statement gets a
  leading comment naming its intent.

```python
# WRONG — dense, semantic boundaries invisible
def acquire(self, timeout: float) -> Connection:
    with self._lock:
        deadline = time.monotonic() + timeout
        while not self._free:
            remaining = deadline - time.monotonic()
            if remaining <= 0:
                raise TimeoutError("no connection available")
            self._cond.wait(remaining)
        conn = self._free.pop()
        self._in_use.add(conn)
        if not conn.is_alive():
            conn = self._create()
            self._in_use.add(conn)
        return conn


# CORRECT — semantic boundaries separated and labelled
def acquire(self, timeout: float) -> Connection:
    with self._lock:
        # Block until a free connection is available or the deadline passes.
        deadline = time.monotonic() + timeout
        while not self._free:
            remaining = deadline - time.monotonic()
            if remaining <= 0:
                raise TimeoutError("no connection available")
            self._cond.wait(remaining)

        # Claim the connection before releasing the lock.
        conn = self._free.pop()
        self._in_use.add(conn)

        # NOTE: a connection may have been evicted by the server while idle;
        #       replace it transparently rather than surfacing a broken handle.
        if not conn.is_alive():
            conn = self._create()
            self._in_use.add(conn)

        return conn
```

The semantic-boundary rule applies even to small transitions. In the example above,
computing `deadline` and waiting on `_cond` are setup; popping from `_free` is a state
mutation — different responsibilities, so they are separated even though both sit inside
the same `while` block's surrounding scope.

---

## 3. Documentation Contracts

### When to write Args / Returns

Write `Args:` only when a parameter's **semantics cannot be inferred** from its name
and type annotation alone. Do not write Args for self-evident parameters.

```python
# WRONG — Args block adds zero information
def set_learning_rate(self, lr: float) -> None:
    """
    Sets the learning rate.

    Args:
        lr (float): The learning rate.
    """

# CORRECT — no Args needed; name + type are sufficient
def set_learning_rate(self, lr: float) -> None:
    """Updates the optimizer learning rate for the current training phase."""
    ...


# CORRECT — Args needed because semantics are non-obvious
def allocate(self, req_id: str, tokens: int) -> bool:
    """
    Reserves physical KV-cache pages for a new or growing request.

    Args:
        req_id (str): Stable identifier for the request across scheduler ticks.
        tokens (int): *Absolute* sequence length, not the delta since last call.

    Returns:
        bool: False on OOM; caller must not proceed with the request.
    """
    # WARNING: OOM pre-screening must precede any state mutation.
    ...
```

### `__init__` specifically

Document `__init__` when constructor parameters have non-obvious semantics or units.
A class docstring describing the object's responsibility is always required.

```python
class PagedKVCacheManager:
    """
    Central manager for the global physical KV-cache block pool.
    Owns allocation state for all active inference requests.
    """

    def __init__(self, pool_size: int, block_size: int) -> None:
        """
        Args:
            pool_size  (int): Total physical blocks across the shared pool.
            block_size (int): Capacity in *tokens* (not bytes) per block.
        """
        ...
```

### File header

Every file opens with a module docstring **before** any imports:

```python
"""
scheduler.py

Owns the token-budget allocation and batch-construction logic for the
inference engine. Interacts with PagedKVCacheManager for memory decisions
and emits BatchDecision objects consumed by GPUEngine._step().
"""

import ...
```

---

## 4. Zone Anchors

Use anchors to segment files with multiple logical sections. Use exactly this format —
no variations in punctuation or width:

```python
# ===========================
# <Zone Name>
# ===========================
```

Zones are separated by **3 blank lines** above the anchor and **1 blank line** below it.

```python
# ===========================
# Memory Allocation
# ===========================

class PagedKVCacheManager:
    ...



# ===========================
# Scheduling Logic
# ===========================

class Scheduler:
    ...
```

---

## 5. CUDA Supplement

All rules above apply. One additional requirement:

**Every pointer parameter in a kernel or host-wrapper must be labelled
`[Device]` or `[Host]` in its docstring.** This is non-negotiable because
passing a host pointer to a kernel produces a silent illegal memory access,
not a compile error.

```cpp
/*
 * Block-level RMSNorm forward pass.
 * Each CUDA block processes one token vector independently.
 *
 * Args:
 *   out (float*)       : [Device] Output buffer for normalized vectors.
 *   in  (const float*) : [Device] Input hidden-state tensor.
 *   w   (const float*) : [Device] Per-channel scale coefficients.
 *   dim (int)          : [Host]   Feature dimension (elements per token).
 *   eps (float)        : [Host]   Stability epsilon; typically 1e-5.
 *
 * NOTE: Block count must equal batch size; grid is 1-D over the token axis.
 */
__global__ void rmsnorm(float* out, const float* in, const float* w,
                        int dim, float eps)
{
    // WARNING: barrier mandatory before warp-reduction; omitting it causes
    //          non-deterministic partial sums across thread groups.
    __syncthreads();
    ...
}
```

---

## Checklist

Run through this before finalising any file:

- [ ] Semantic tokens used for all non-obvious constraints; no `!!!` or `IMPORTANT`.
- [ ] Semantic boundaries and logical phases inside functions separated by blank lines and labelled.
- [ ] 2 blank lines between methods; 3 before zone anchors.
- [ ] Long argument lists broken across lines with trailing comma.
- [ ] Class docstring present on every class.
- [ ] `Args:` written only where semantics are genuinely non-obvious.
- [ ] File-level docstring present before imports.
- [ ] *(CUDA only)* Every pointer parameter carries `[Device]` or `[Host]`.
