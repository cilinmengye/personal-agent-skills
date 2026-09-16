# Physical Structure Review

Use this reference during the Explore phase. It is a decision guide for the physical organization of code: files, directories, packages, build targets, tests, and public entry points.

## The central distinction

A **module** is a behavior-bearing abstraction with an interface and an implementation. A file or directory is only a physical container. A deep module may have one small public entry point and many cohesive private files.

Splitting a large file is useful when it improves ownership, locality, dependency direction, or navigation while preserving a small interface. Merely distributing the same tangle across more files creates fragmentation.

## Principles

- **Ownership before proximity.** Place behavior with the module that owns its invariants and lifecycle, rather than with the currently open file or nearest generic directory.
- **Change together, stay together.** Code that changes for one reason belongs close together. Independently changing behavior deserves a distinct private role or owner.
- **Public depth, private legibility.** Keep the caller-facing interface small. Organize implementation files so maintainers can find and change one concept locally.
- **Real boundaries before visual symmetry.** Independent build, dependency, release, runtime, deployment, process, security, public-consumption, or durable ownership lifecycles justify strong package boundaries. A language or toolchain difference counts only when it creates one of those independent lifecycles; language alone is insufficient. A balanced-looking tree does not.
- **One primary navigation axis.** Let paths express the repository's most stable capability or ownership dimension. Put orthogonal dimensions such as hardware variant or test cost in metadata or configuration when nesting them would create a directory Cartesian product.
- **Evidence before movement.** Imports, manifests, tests, change history, conflicts, and recurring bugs are evidence. File length is an invitation to inspect cohesion, not a verdict.
- **Tests follow behavioral ownership.** Organize tests around public behavior and the subsystem that owns it, even when the repository keeps all tests under a separate top-level tree.

## Evidence to inspect

Use the smallest set that can support or falsify a candidate:

- nearby and top-level directory trees;
- repository instructions, architecture documents, `CONTEXT.md`, and relevant ADRs;
- package manifests, build targets, code-generation rules, and distribution metadata;
- public entry points, exports, imports in both directions, and dependency cycles;
- representative tests, fixtures, and test-discovery configuration;
- recent history, co-change patterns, repeated conflicts, and frequently touched paths;
- `CODEOWNERS` or equivalent ownership metadata;
- runtime, deployment, language, hardware, and performance constraints.

Treat history as a prioritization signal, not proof. A recent rename, generated file, or broad formatting commit can create misleading co-change patterns.

## Friction to look for

- **Append gravity:** new behavior repeatedly lands in an already-open file even though it has a different lifecycle or dependency set.
- **Ownership fog:** `core`, `common`, `utils`, `manager`, or a similar container accumulates unrelated capabilities because no clearer owner is named.
- **Hotspot mixing:** one file changes for independent features, contributors, or operational reasons and repeatedly attracts conflicts.
- **Navigation tax:** understanding one capability requires searching unrelated regions or bouncing through decorative one-function files.
- **Public-surface drift:** internal details become exports, or every implementation addition forces callers to learn another entry point.
- **Dependency leakage:** reverse imports, cycles, cross-subsystem shortcuts, or framework and vendor details escape their intended seam.
- **Boundary mismatch:** an independent build, dependency, release, runtime, deployment, process, security, public-consumption, or ownership lifecycle is hidden inside a directory that implies one homogeneous module.
- **Test distortion:** production helpers become public only for tests, or a behavior's tests require unrelated fixtures because responsibilities are mixed.
- **Structure metadata drift:** manifests, generated-file rules, test discovery, documentation, or ownership rules no longer match the paths they govern.
- **Composition-root overload:** an object that should construct, wire, delegate, and coordinate also performs configuration calculations, transformations, or algorithms. Knowing many collaborators is normal; doing their work is not. Extracted collaborators should receive narrow inputs and return results rather than inherit the whole god object.
- **Extension pressure:** a third comparable implementation, a growing central conditional, heavy optional dependencies, or external integration pressure suggests a registry or plugin seam. A variant count is a prompt to inspect the selection problem, not an automatic command to add indirection.
- **Naming drift:** a path such as `core`, `common`, or an obsolete domain term no longer communicates the capability that owns the code, even when the code is otherwise in the right physical location.

## Placement ladder and structural moves

Choose the smallest move that resolves the observed friction. Every candidate must state:

```text
Chosen structural level:
Why this level is sufficient:
Why one level smaller is insufficient:
Why one level larger is excessive:
```

For merge, relocation, and rename candidates, interpret the adjacent levels as the weaker and broader alternatives: explain why a smaller correction would leave the friction in place and why a broader boundary change would be excessive.

Use this ladder for changes that introduce stronger physical boundaries:

1. **Keep the existing file.** The behavior shares the file's capability, invariants, dependencies, lifecycle, and reason to change. A single-consumer helper normally stays private and close to its owner.
2. **Add a private implementation file.** At least one observed signal exists: an independently named role, state or invariant, dependency set, or reason to change. The split improves locality, leaves the owning file closer to orchestration, and lets a collaborator work through narrow inputs and returned results without adding a public interface.
3. **Create a private subpackage.** Several cohesive implementation roles already form one named capability with a common owner, one intentional facade, and an internal dependency direction. Its own test family or extension family is supporting evidence. Do not create a directory for one file or a hypothetical future taxonomy.
4. **Create a top-level package, repository area, or service.** A genuinely independent build, dependency set, release cadence, deployment unit, process or security boundary, public-consumption contract, or durable maintenance owner requires it. A different language or toolchain is evidence only when it creates one of those boundaries.

Use these lateral moves when a stronger container is not the answer:

- **Split a hotspot.** Independently changing responsibilities, dependency sets, states, or test setups already coexist and have credible target owners.
- **Merge shallow fragments.** Files or packages share one lifecycle and consumer, change together, leak state across seams, and force readers to reconstruct one concept without hiding useful complexity.
- **Relocate misplaced behavior.** An existing capability clearly owns the behavior and the move restores dependency direction or domain locality.
- **Rename a misleading path.** The owner is correct but the name no longer expresses the project's domain model. Prefer a mechanical rename with compatibility handling where public imports are involved.

Every proposed file must correspond either to responsibilities, dependencies, or code observed in the repository, or to a responsibility, dependency, runtime, or lifecycle explicitly introduced by the user's spec or plan. Label the latter `Prospective`. Ground it in both the stated change and the repository's current seams; a prospective candidate is not permission to prebuild a taxonomy. Mark uncertain cuts as hypotheses and inspect the relevant implementation before promoting them to candidates.

## Candidate tests

Apply the relevant tests; no single test is universal.

- **Ownership test:** Which module owns the behavior's invariants and lifecycle? If the answer is vague, ownership is the first design problem.
- **Placement-level test:** What is the smallest structural level that resolves the friction, and what concrete evidence rules out the adjacent smaller and larger levels?
- **Change-coupling test:** Do these parts change together for one reason, or only appear together because of the current layout?
- **Locality test:** Will the proposed structure concentrate the knowledge, bugs, and verification for a capability?
- **Deletion test:** For a suspected shallow module, would deleting it concentrate complexity or merely move complexity into every caller? Concentration supports a merge or deepening candidate.
- **Split counterfactual:** If the proposed new files vanished, would their responsibilities collapse back into one mixed owner? If nothing meaningful changes except filenames, the split is decorative.
- **Interface test:** Can callers continue through one small interface while the implementation becomes easier to navigate?
- **Dependency-direction test:** Can the intended direction be stated and mechanically checked without cycles or shortcuts?
- **Boundary test:** Does the proposed package boundary correspond to an independent operational, build, dependency, release, deployment, process, security, public-consumption, or ownership lifecycle?
- **Test-surface test:** Can behavior be verified through the owning module's interface, with private seams remaining private?
- **Composition-root test:** After the move, does orchestration construct, wire, delegate, and coordinate while named collaborators own calculation and policy through narrow inputs?
- **Extension-seam test:** Is there a recurring selection or discovery problem with enough real variants to justify a registry or plugin seam, rather than a speculative abstraction?
- **Naming test:** Do the proposed paths use the project's current domain language and make ownership more legible without moving otherwise well-owned behavior?

## Candidate quality and structural success gate

A reportable candidate contains:

1. observed friction with concrete paths or dependency evidence, or a prospective structural pressure caused by an explicit upcoming change;
2. a causal explanation connecting the current structure to that friction;
3. a credible current or proposed owner;
4. a bounded before/after tree or dependency sketch;
5. the intended public entry point and private implementation area;
6. affected tests, builds, exports, generated rules, and ownership metadata;
7. migration risk, counterevidence, and a confidence label;
8. the chosen structural level, why it is sufficient, and why the adjacent smaller and larger levels are wrong;
9. a **Structural Success Contract**.

The Structural Success Contract contains:

```text
Friction to remove:
Expected locality/interface/dependency/navigation/ownership/testability outcome:
How each claimed improvement will be verified:
Architecture enforcement, or why none is justified:
Behavior-preservation checks:
Runtime-sensitive mechanisms touched: none / list
Conditional runtime validation:
```

A candidate must name at least one material, verifiable structural improvement; it need not improve every dimension. It must not buy a cleaner-looking tree by creating shallow modules, enlarging the public surface, weakening locality, or adding unexplained navigation. Existing behavior checks are preservation guardrails, not evidence that the new structure is better.

Runtime validation is conditional. Require it only when the structural move changes import or loading behavior, dynamic dispatch, registry or plugin discovery, process or IPC boundaries, GPU hot paths or synchronization, package/build loading, or another identified runtime-sensitive mechanism. A mechanical move with none of these pressures records `none` rather than demanding a benchmark.

**A prettier directory tree is not a benefit by itself. A structural change succeeds only when it makes change more local, the interface more intentional, dependencies clearer, ownership more accurate, navigation more meaningful, or testing more natural, without an unacknowledged structural regression.**

Use these recommendation strengths:

- **Strong:** recurring/current friction or imminent pressure from an explicit approved change, plus a clear structural cause, a credible target owner, and a bounded migration.
- **Worth exploring:** current friction or well-grounded prospective pressure is visible, but ownership, seam placement, or migration cost needs discussion.
- **Speculative:** a plausible shape with weak evidence or unclear payoff. Preserve it as a hypothesis rather than queued work.

An audit with no `Strong` or `Worth exploring` candidate is a valid result. Report why the inspected structure is adequate and which signals would justify looking again.

Separately label candidate timing as `Current` or `Prospective`. Strength expresses confidence and payoff; timing expresses whether the pressure exists today or follows from an explicit planned change.

## Migration shape

For a substantial relocation, plan separately verifiable stages:

1. **Prepare:** narrow dependencies and establish the target interface while behavior stays in place.
2. **Mechanical move:** move or rename with the smallest semantic delta and verify structural equivalence where practical.
3. **Postpare:** remove compatibility paths, then make the intended design or behavior changes.

A side-by-side transition is appropriate when each intermediate state builds, runs, and has one declared owner. Compatibility paths need an exit condition so two permanent owners do not remain.

## Common false positives

- A large cohesive file can be easier to understand than many small public files.
- Unequal directory sizes may accurately reflect unequal domain complexity.
- Generated or vendored code follows provenance and build constraints different from handwritten code.
- A one-time burst of changes may not indicate a durable hotspot.
- A future taxonomy without current code or dependencies is speculation, not architecture.
