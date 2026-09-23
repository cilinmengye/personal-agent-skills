---
name: test-strategy
description: Choose contract-based test levels and cost-aware execution for expensive systems, refactor-driven test migration, fault injection or test doubles, and white-box lifecycle, concurrency, or performance contracts.
license: MIT
---

# Test Strategy

Choose the evidence needed to protect an accepted behavior and obtain it at the lowest sufficient cost. Preserve the project's behavior, interfaces, acceptance gates, and test policy. Product-interface design, red-green sequencing, implementation, and final review remain with their owning workflows.

When TDD is active, TDD owns the confirmed seam and red → green sequence. This skill explains what each test level proves, which evidence remains missing, and when a more expensive level is required. It does not invoke another skill automatically.

## Establish the evidence contract

Before adding a non-trivial test group or migrating tests that share a seam and runtime, state one compact contract:

```text
Protected behavior:
Independent expected result:
Chosen seam and test level:
Evidence provided:
Evidence still missing:
Evidence invalidators:
Final validation set:
```

Keep each field to one line unless a safety or performance contract needs a short list. An independent expected result may come from a specification example, known literal, reference implementation, invariant, or metamorphic relation; it must be able to disagree with the production logic. Reuse an unchanged contract across related cases. For a mechanical rename, type-only change, or wiring edit with no new independently observable behavior, state that no new test is needed and list the existing checks that cover the risk.

This step is complete when the protected behavior, independent oracle, lowest sufficient level, invalidators, and final validation set are explicit.

## Select the lowest sufficient level

Choose the cheapest level that exercises the real behavior under test:

- Run production objects directly for pure logic, state transitions, scheduling, accounting, and deterministic data transformations.
- Use the smallest real integration for cross-module contracts, protocols, transactions, concurrency, and resource lifecycles.
- Use real processes, IPC, GPU execution, model inference, service protocols, or browsers when the protected behavior exists at that level.
- Run a full model matrix or performance measurement when a corresponding contract, runtime mechanism, or measured hot path is affected.

The repository owns test placement and commands. This skill owns the evidence choice and execution scope.

## Choose a durable seam

A durable test seam is either a caller-facing interface or an intentionally maintained internal seam with a named owner and contract.

Use private fields, object-layout assertions, constructor bypasses, or call-order checks only when they protect an explicit lifecycle, safety, concurrency, resource, or performance contract that a coarser interface cannot prove. Record the expected migration cost. When the accepted specification and project policy have not already selected them, confirm with the user before creating a new durable seam or an expensive validation matrix.

When the module interface itself is unsettled, use the project's interface-design workflow. Keep this skill focused on evidence and cost.

## Bound test doubles and fault injection

Keep the production logic named in `Protected behavior` real. A test double may replace an external or expensive dependency that sits outside that behavior when project policy permits it.

For each replacement, name the evidence it excludes and the real integration that supplies any required missing evidence. Known tokens driving a real output processor prove output processing. They leave model generation unproven.

Describe fault injection by the path it exercises. An injected ordinary exception can prove propagation and cleanup for that exception path. Claims about CUDA faults, forced process termination, hardware failure, or transport loss require evidence from the corresponding mechanism.

## Execute by evidence invalidation

Run in this order:

1. the current failing case, when one exists, and directly related checks;
2. the lowest sufficient real integration after a protocol, resource, concurrency, process, or hardware contract changes;
3. the predeclared final validation set before completion.

Treat source builds, loaded process code, dependencies, configuration, and persistent mutable state as part of the evidence. A change to any relevant part invalidates prior results at the affected level.

When a failure repeats without new evidence, stop rerunning it and diagnose the cause. Assertion weakening, arbitrary waits, and retries do not create evidence.

For model, GPU, browser, service, multiprocess, startup, shutdown, or destructive failure tests, read [references/EXPENSIVE-RUNS.md](references/EXPENSIVE-RUNS.md) before running them.

For tests affected by a module extraction, ownership move, interface change, or directory restructuring, read [references/TEST-MIGRATION.md](references/TEST-MIGRATION.md) before editing the tests.

## Finish with an evidence ledger

Finish when every affected behavior has a named evidence source, every migrated test has a disposition, invalidated results have been rerun, and the final validation set has completed. Record expensive commands, environment restarts, final results, and uncovered contracts. Describe each passing test only within the evidence scope established above.

If an active implementation or review workflow accepts a code change after testing, rerun the levels invalidated by that change. Review invocation and finding remediation remain with that workflow.
