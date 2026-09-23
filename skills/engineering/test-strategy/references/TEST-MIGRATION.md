# Test Migration During Structural Change

Use this reference when code ownership, module interfaces, directories, packages, or collaborators move while accepted behavior is intended to remain stable.

## Inventory the protected contracts

For every affected test, identify the behavior it protects, its seam, and whether its expected result comes from an independent source. Then assign exactly one disposition:

- **Keep:** the behavior and seam remain stable.
- **Migrate:** the behavior remains stable and its owned seam has moved.
- **Replace:** a higher-level contract now proves the same behavior with equal or stronger evidence.
- **Retire:** the behavior contract has been intentionally removed, with a cited decision or specification.
- **Preserve white-box:** an explicit lifecycle, safety, concurrency, resource, or performance contract still requires internal evidence at the new owner.

## Preserve evidence through the move

Move valid behavior assertions to the new owner or interface. Prefer the new supported construction path. Rebuilding removed private state solely to preserve an old fixture creates evidence for an obsolete structure.

When a white-box test remains necessary, record the contract that justifies it, why a coarser seam cannot prove it, and the expected maintenance cost. Keep the assertion limited to that contract.

Delete a test only after its contract is covered by the replacement evidence or formally retired. A passing higher-level test counts as a replacement only when it exercises the same failure mode and observable outcome.

The migration is complete when every affected test has one disposition, every retained contract has independent evidence, obsolete construction scaffolding has been removed, and the final validation set covers the new structure.
