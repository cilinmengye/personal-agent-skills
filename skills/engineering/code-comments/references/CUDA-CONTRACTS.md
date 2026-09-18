# CUDA contract fields

Use this reference only for a public kernel, public host wrapper, or cross-module CUDA seam when its types do not already make the contract explicit. Follow any stronger repository-specific schema.

## Memory and data

For every buffer whose location matters, record the actual memory space, such as `[Device global]`, `[Device shared]`, `[Host pinned]`, `[Host pageable]`, `[Managed]`, or `[Mapped]`. Avoid reducing managed or mapped memory to a binary Host/Device label.

Record only fields that constrain a caller or maintainer:

- read, write, or read/write access;
- allocator/owner, transfer of ownership, and valid lifetime;
- permitted aliasing and overlap;
- shape, stride, dtype, units, and alignment;
- initialization or residency requirements.

Scalar values have no pointer memory space. Document their units, valid range, sentinel meaning, or relationship to tensor dimensions instead.

## Execution

Record execution constraints that are not enforced by the wrapper or launch configuration:

- grid and block dimensional assumptions;
- dynamic shared-memory requirements;
- stream association and whether the operation is asynchronous;
- events, barriers, prior work, or synchronization required before reuse;
- permitted devices, architectures, and capture contexts;
- required call ordering and error-observation point.

## Performance claims

Use `PERF:` only for an assumption that is measured or forms part of the supported performance contract. State the affected architecture or workload and point to the benchmark, profile, or regression test when one exists. Do not turn a guessed launch choice into a documented guarantee.

## Compact example

```cpp
/**
 * Enqueues an in-place compaction of active rows on `stream`.
 *
 * @param rows [Device global, read/write] Contiguous fp16 matrix with
 *     `row_count * stride` elements. Owned by the caller and valid until the
 *     stream completes. Must not alias `indices`.
 * @param indices [Device global, read] int32 source rows, valid for
 *     `row_count` elements on the same device as `rows`.
 * @param row_count Logical rows in `rows`; non-negative.
 * @param stride Elements per row; a multiple of 8 for the vectorized path.
 * @param stream Work is enqueued asynchronously; caller synchronizes before
 *     reading or freeing either buffer.
 *
 * PERF: The vectorized path is guarded by the stride/alignment checks and is
 * validated by the repository's compaction benchmark.
 */
void compact_rows(
    half* rows,
    const int32_t* indices,
    int row_count,
    int stride,
    cudaStream_t stream);
```
