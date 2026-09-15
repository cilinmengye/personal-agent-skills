# Physical Structure Review

Use this reference during the Explore phase. It is a decision guide for the physical organization of code: files, directories, packages, build targets, tests, and public entry points.

## The central distinction

A **module** is a behavior-bearing abstraction with an interface and an implementation. A file or directory is only a physical container. A deep module may have one small public entry point and many cohesive private files.

Splitting a large file is useful when it improves ownership, locality, dependency direction, or navigation while preserving a small interface. Merely distributing the same tangle across more files creates fragmentation.

## Principles

- **Ownership before proximity.** Place behavior with the module that owns its invariants and lifecycle, rather than with the currently open file or nearest generic directory.
- **Change together, stay together.** Code that changes for one reason belongs close together. Independently changing behavior deserves a distinct private role or owner.
- **Public depth, private legibility.** Keep the caller-facing interface small. Organize implementation files so maintainers can find and change one concept locally.
- **Real boundaries before visual symmetry.** Build, distribution, language, runtime, deployment, hardware, public-consumption, and durable ownership boundaries justify strong package boundaries. A balanced-looking tree does not.
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
- **Boundary mismatch:** a build, runtime, deployment, language, hardware, or ownership boundary is hidden inside a directory that implies one homogeneous module.
- **Test distortion:** production helpers become public only for tests, or a behavior's tests require unrelated fixtures because responsibilities are mixed.
- **Structure metadata drift:** manifests, generated-file rules, test discovery, documentation, or ownership rules no longer match the paths they govern.

## Structural moves

Choose the smallest move that resolves the observed friction.

1. **Keep the existing file.** The behavior shares the file's capability, invariants, dependencies, lifecycle, and reason to change.
2. **Add a private implementation file.** A concrete internal role or dependency set already exists and separating it improves locality without adding a public interface.
3. **Create a private subpackage.** Several observed implementation roles form one named capability, benefit from an internal dependency direction, and can remain behind one intentional entry point.
4. **Create a top-level package or repository area.** A real build, distribution, language, runtime, deployment, hardware, public-consumption, or durable ownership boundary requires it.
5. **Split a hotspot.** Independently changing responsibilities, dependency sets, or test setups already coexist and have credible target owners.
6. **Merge shallow fragments.** Files or packages share one lifecycle and consumer, always change together, leak state across seams, and make readers reconstruct one concept without hiding useful complexity.
7. **Relocate misplaced behavior.** An existing capability clearly owns the behavior and the move restores dependency direction or domain locality.

Every proposed file must correspond either to responsibilities, dependencies, or code observed in the repository, or to a responsibility, dependency, runtime, or lifecycle explicitly introduced by the user's spec or plan. Label the latter `Prospective`. Ground it in both the stated change and the repository's current seams; a prospective candidate is not permission to prebuild a taxonomy. Mark uncertain cuts as hypotheses and inspect the relevant implementation before promoting them to candidates.

## Candidate tests

Apply the relevant tests; no single test is universal.

- **Ownership test:** Which module owns the behavior's invariants and lifecycle? If the answer is vague, ownership is the first design problem.
- **Change-coupling test:** Do these parts change together for one reason, or only appear together because of the current layout?
- **Locality test:** Will the proposed structure concentrate the knowledge, bugs, and verification for a capability?
- **Deletion test:** For a suspected shallow module, would deleting it concentrate complexity or merely move complexity into every caller? Concentration supports a merge or deepening candidate.
- **Split counterfactual:** If the proposed new files vanished, would their responsibilities collapse back into one mixed owner? If nothing meaningful changes except filenames, the split is decorative.
- **Interface test:** Can callers continue through one small interface while the implementation becomes easier to navigate?
- **Dependency-direction test:** Can the intended direction be stated and mechanically checked without cycles or shortcuts?
- **Boundary test:** Does the proposed package boundary correspond to a real operational, build, distribution, language, or ownership fact?
- **Test-surface test:** Can behavior be verified through the owning module's interface, with private seams remaining private?

## Candidate quality gate

A reportable candidate contains:

1. observed friction with concrete paths or dependency evidence, or a prospective structural pressure caused by an explicit upcoming change;
2. a causal explanation connecting the current structure to that friction;
3. a credible current or proposed owner;
4. a bounded before/after tree or dependency sketch;
5. the intended public entry point and private implementation area;
6. affected tests, builds, exports, generated rules, and ownership metadata;
7. migration risk, counterevidence, and a confidence label.

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
- A composition root is expected to know many collaborators; its job should remain construction, wiring, delegation, and coordination.
- A third implementation can justify a registry or plugin seam, while one implementation rarely does.
- A one-time burst of changes may not indicate a durable hotspot.
- A future taxonomy without current code or dependencies is speculation, not architecture.
