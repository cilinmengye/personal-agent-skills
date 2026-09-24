# Test Migration During Structural Change

Use this reference when code ownership, module interfaces, directories, packages, or collaborators move while accepted behavior is intended to remain stable.

## Group affected tests by protected contract

For every affected test, record its protected behavior, failure mode, seam, and independent expected result. Group tests with equivalent behavior and failure modes; compare their seams and expected results to retain distinct evidence. Separate cases that protect a distinct risk or runtime mechanism, even when their final output matches another group. Identify accepted contract changes before choosing a disposition. Decide the disposition for each group and account for its member tests; the replacement test count need not match the old count.

- **Keep:** the behavior and seam remain stable.
- **Migrate:** the behavior remains stable and its owned seam has moved.
- **Consolidate:** multiple tests provide equivalent evidence for the same failure mode. Preserve distinct scenarios and clear failure locations; use parameterization when it preserves both.
- **Replace:** a test at a more suitable seam proves the same failure mode and observable outcome with equivalent evidence.
- **Remove change detector:** the test mirrors incidental private layout or interactions and protects no accepted behavior or invariant. Check its cases for unique risks, and state what behavioral evidence remains or why its assertions never supplied such evidence.
- **Update or retire:** the accepted behavior contract has changed or ended. Cite the decision or specification and update surviving assertions.
- **Preserve white-box:** an explicit lifecycle, safety, concurrency, resource, or performance contract still requires internal evidence at the new owner.

## Preserve evidence through the move

Move valid behavior assertions to the new owner or interface. Prefer the new supported construction path. Rebuilding removed private state solely to preserve an old fixture creates evidence for an obsolete structure. Add focused tests for newly introduced process, thread, or resource-ownership contracts when the existing evidence does not exercise those mechanisms.

When a white-box test remains necessary, record the contract that justifies it, why a coarser seam cannot prove it, and the expected maintenance cost. Keep the assertion limited to that contract.

Remove a test after its contract has equivalent replacement evidence, has formally ended, or has been shown to contain only an implementation-shape check. A passing test at another level counts as a replacement only when it exercises the same failure mode and observable outcome. Keep a targeted internal check when a broad successful run cannot reveal a safety or lifecycle violation.

The migration is complete when every affected test belongs to a group with a disposition, every retained behavior and distinct failure mode has independent evidence, obsolete construction scaffolding has been removed, and the final validation set covers the new structure.
