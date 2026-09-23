---
name: write-maintainable-code
description: Choose and write the smallest maintainable implementation after behavior, interfaces, and scope are already decided.
disable-model-invocation: true
license: MIT
---

# Write Maintainable Code

Guide the implementation itself after the requested behavior, public seam, and task boundary are settled. Preserve those decisions; do not restart discovery, redesign the product, choose test seams, or perform a final code review.

Use this for human-maintained production and test code. Generated code, vendored code, and intentionally throwaway prototypes follow their own lifecycle instead.

## Before a non-trivial implementation edit

Read the repository instructions and the smallest surrounding area needed to understand the owner, invariants, existing conventions, dependencies, callers, and error/resource lifecycle. Search for an existing capability before adding one.

Then state this compact implementation choice before the first non-trivial implementation edit:

```text
Owner and invariant:
Representation and reuse:
Necessary local prefactor: none / list:
Lifecycle risks and ownership: only when applicable
Portability/performance evidence: only when applicable
```

Here, a **prefactor** is a bounded, behavior-preserving preparatory refactor required to express the current change. A mechanical rename, formatter-only edit, or obvious one-line correction may skip the brief. For other work, complete the three always-applicable lines and include each conditional line only when its trigger exists. Every included line must name a concrete choice rather than a slogan.

When TDD is active, TDD owns sequencing: confirm the seam and establish the failing red test first, then state this brief before the first non-trivial green implementation edit. A prefactor may be part of green only when the current behavior cannot be expressed without it. Defer other refactoring and implementation-quality remediation until the red-green loop is complete.

## Decision gates

### Simplicity is a reasoning property

Measure simplicity by the number of concepts, branches, dependencies, hidden state transitions, and public choices a maintainer must hold at once—not by line count. Keep one honest owner for each invariant. Accept internal complexity when it hides necessary detail behind a smaller, more stable interface.

### Classify apparent hard-coding

Place a value according to who owns its change:

- A stable domain invariant stays in code with a domain name and a focused behavioral check.
- A deployment-specific value belongs in configuration supplied at the deployment boundary.
- A user or product choice is an explicit policy or input, not a literal buried in mechanism code.
- An implementation detail can remain local while there is one real behavior. Introduce a variation seam when a second real behavior or a known external contract proves the axis.

Do not turn every literal into configuration. The goal is to localize each decision at its real owner.

### Reuse in cost order

Prefer, in order:

1. a suitable capability already owned by the project;
2. the language or platform standard library;
3. an already-approved project dependency;
4. a small local implementation whose maintenance cost is clear;
5. a new dependency only after its API fit, maintenance, security, build, size, and portability costs are justified.

Reuse behavior and established knowledge. Avoid thin wrappers that merely rename an API or a dependency added to replace a few stable, obvious operations.

### Earn abstractions with evidence

An abstraction must do at least one of these:

- hide a real variation or platform boundary;
- keep an invariant or piece of knowledge in one owner;
- make callers materially simpler;
- replace a repeated control-flow shape that is already changing together.

Prefer a direct implementation for a single behavior. When a genuine second behavior appears, compare their stable common contract and varying policy before choosing a function, data representation, strategy, registry, or other seam. A second behavior earns evaluation of a seam, not an automatic public interface; keep the seam local unless the settled public contract requires exposure. Do not create a framework for hypothetical variants.

### Make state and control flow visible

Prefer explicit inputs, results, state transitions, and locally checkable invariants. As repeated branches grow, ask whether a table, data representation, or named state model makes the permitted cases clearer. Keep orchestration readable and move owned detail only when the new collaborator receives narrow data rather than the entire mutable context.

### Give effects and failure a clear owner

Keep I/O, mutation, resource acquisition/release, retries, and error translation at an identifiable boundary. Make partial failure and cleanup understandable without tracing unrelated modules. Preserve the repository's error model; add handling for reachable failures, not imagined impossible states.

When the change touches resource acquisition, asynchronous work, shared state, or concurrent shutdown, read [references/RESOURCE-LIFECYCLE.md](references/RESOURCE-LIFECYCLE.md) before implementation. Use it to complete `Lifecycle risks and ownership` with the affected state transitions, ownership transfers, stop confirmation, and failure outcomes, then account for every reachable lifecycle path affected by the change.

### Default to a portable baseline

Choose the clearest portable implementation unless the specification contains a performance contract or measurements identify a hot path. When specialization is justified, isolate it behind a narrow boundary and name the evidence and invariants the optimized path depends on. Retain a clear baseline when the project supports platforms outside the specialization, needs an independent correctness reference, or selects implementations at runtime. Python/C++/CUDA code may use platform-specific mechanisms when platform specificity is part of the requirement.

For a measured hot path, complete `Portability/performance evidence` with the representative workload, device or platform, relevant toolchain versions, baseline, measurable accepted target, measurement method, and specialized-path assumptions. Reuse the project's benchmark protocol where one exists. After implementation, record the measured result and compare it with the baseline and target under the same protocol.

### Allow a necessary local prefactor

A prefactor is justified when it is directly needed to express the current behavior, preserves behavior, and has a bounded diff—for example, naming an existing concept, exposing the required seam, or consolidating the state transition being changed. Keep unrelated cleanup outside the task and report it separately.

## Write the implementation

Implement the chosen representation with the smallest public surface that satisfies the settled behavior. Match repository naming, typing, error, dependency, formatting, and testing conventions. Keep related knowledge local, and remove only the orphaned code created by this change.

This skill does not automatically invoke other skills. When the user also invokes a planning, TDD, comment, structure, or review skill, keep this skill confined to implementation-internal choices. The orchestrating skill owns workflow order; in particular, TDD owns red-green sequencing and final code review owns independent Standards/Spec findings.

## Implementation-quality check

Run one bounded pass over only the code written or materially changed by this implementation:

- Is every literal owned as an invariant, deployment value, product policy, or local implementation detail?
- Did each new abstraction, dependency, and configuration option pass its gate?
- Are state transitions, effects, cleanup, and reachable failure paths locally understandable?
- Does specialized code have evidence, isolation, and stated assumptions?
- Was every prefactor necessary, behavior-preserving, and bounded?
- Could the same behavior be expressed with fewer concepts or a smaller public surface without hiding important constraints?

Fix evidence-backed implementation-local issues within the active workflow's remediation stage. When a fix materially changes the implementation choice, recheck only the affected gate. Finish when the accepted contract is satisfied, the necessary local issues are resolved, and the active workflow's relevant verification has passed. When TDD is active, preserve red → minimal green before implementation-quality remediation. Specification validation, test-level selection, repository-wide structure work, and independent review remain with their owning workflows.
