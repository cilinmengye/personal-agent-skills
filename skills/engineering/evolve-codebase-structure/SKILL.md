---
name: evolve-codebase-structure
description: Survey a codebase's physical organization, present evidence-backed restructuring candidates, and shape one through a decision interview.
---

# Evolve Codebase Structure

Run a periodic, read-first architecture survey focused on physical code organization: where behavior belongs, which structural level is the smallest sufficient move, when files should merge, and when a real package boundary has emerged.

This skill queues and shapes work; it does not refactor source code. Its first two phases are read-only except for one HTML report in the OS temp directory. The grilling phase may update domain documentation when a decision genuinely changes the domain model, but implementation belongs to a later build session.

Use `codebase-design` for the shared vocabulary of module, interface, implementation, depth, seam, adapter, leverage, and locality. Read existing `CONTEXT.md` and relevant ADRs for the project's language and settled decisions.

## Inputs

Use what the user supplies:

- an optional subsystem, pain point, upcoming spec, or other focus;
- an optional history baseline such as a commit, date, or release;
- an optional previous structure-review HTML file.
- an optional `report-only` request that ends after Phase 2.

An explicit focus takes priority. Without one, infer a bounded scope from a meaningful stretch of recent history and active structural hotspots. A previous report turns the run into a delta review: mark candidates as new, changed, carried forward, resolved, or rejected for a durable reason.

## Process

### 1. Explore physical structure

Read [STRUCTURE-DESIGN.md](STRUCTURE-DESIGN.md) before scanning.

Map the relevant top-level and nearby trees, public entry points, imports in both directions, tests, manifests, build and generated-file rules, ownership metadata, and recent changes. Use a fresh exploration sub-agent when the environment supports one; otherwise perform the same evidence pass directly. Keep this phase read-only.

Explore organically, while giving extra attention to:

- code that grows by repeatedly appending to the current file;
- files mixing independently changing capabilities, lifecycles, or dependency sets;
- behavior whose owner is obscured by `core`, `common`, `utils`, or similar containers;
- fragmented modules that impose navigation without hiding complexity;
- misplaced behavior, reverse imports, cycles, and cross-subsystem shortcuts;
- composition roots doing calculation or policy, collaborators receiving a god object, and central conditionals accumulating real variants;
- directory names that no longer express the project's domain model;
- build, runtime, language, deployment, hardware, or ownership facts hidden by the directory tree;
- tests, exports, and generated rules distorted by the present layout.

Treat line count as a prompt to inspect cohesion. Apply the placement ladder, candidate tests, and Structural Success Contract in `STRUCTURE-DESIGN.md`; record counterevidence and discard decorative splits. For each candidate, choose the smallest sufficient structural move and explain why the adjacent smaller and larger levels are wrong. For an upcoming change, admit `Prospective` candidates only when the user's stated responsibility, dependency, runtime, or lifecycle creates a concrete pressure against the repository's present seams. Preserve uncertain ideas as hypotheses with `Speculative` strength.

The exploration is complete when the bounded scope and evidence sources are explicit, each inspected hotspot is either a candidate or a rejected false positive with a reason, and every candidate has selected its smallest sufficient move and passes the structural success gate in `STRUCTURE-DESIGN.md`. Zero actionable candidates is a valid result.

### 2. Present candidates as an HTML report

Read [HTML-REPORT.md](HTML-REPORT.md), then create the report it specifies in the OS temp directory. Open it for the user and state its absolute path.

Rank candidates by expected structural payoff, current relevance, confidence, and migration risk. Put at most five principal candidates in the main report; keep weaker hypotheses and rejected false positives in the appendix. Every card must separate observation from inference, name the chosen structural level and rejected adjacent levels, show a compact current versus possible tree or dependency view, and state a verifiable success hypothesis. Describe a provisional interface hypothesis only far enough to make the structural move intelligible; reserve the final interface and unresolved ownership choices for discussion.

The report phase is complete when every candidate is traceable to concrete evidence, counterevidence is visible, each success hypothesis can be checked by a downstream build, the top recommendation is justified, and the report can honestly say that no structural change is warranted.

Branch at the end of this phase:

- With `report-only`, finish after presenting the report path and verdict.
- Otherwise, with at least one `Strong` or `Worth exploring` candidate, ask which candidate ID the user wants to explore.
- With only `Speculative` candidates, give a no-change recommendation and ask whether the user intentionally wants to explore one.
- With no candidate, finish with the no-change verdict and revisit signals; the grilling phase does not run.

When the run continues into grilling, wait for a selection first. A `report-only` run completes immediately after the report is presented. Source code remains unchanged.

### 3. Grill the selected candidate

Resume from one selected report card rather than repeating the scan or grilling several candidates in one context. Use `grilling` to work a design tree in rounds. Environmental facts are the agent's responsibility; decisions remain with the user. Use `domain-modeling` when terms or durable architectural decisions genuinely crystallize, and use `codebase-design` when choosing a seam or comparing alternative interfaces.

Cover the branches that apply:

- capability name, owner, invariants, lifecycle, and scope;
- the chosen structural level, why it is sufficient, and why the adjacent smaller and larger levels are wrong;
- the current and target tree, including each proposed file's concrete responsibility;
- one intentional caller-facing surface, private implementation roles, narrow collaborator inputs, and permitted dependency direction;
- keep, split, merge, relocation, rename, or package-boundary alternatives, including the do-nothing option;
- the Structural Success Contract: friction removed, expected structural improvement, verification, preservation guardrails, and enforcement;
- prepare, mechanical-move, and postpare stages or another buildable migration path;
- tests, manifests, exports, generated-file rules, documentation, and ownership metadata;
- compatibility, rollback, deployment, and hardware constraints, plus runtime validation only when an identified runtime-sensitive mechanism changes;
- enforceable architecture checks and the conditions under which the structure should be revisited.

If radically different interface designs would clarify the choice, use the `codebase-design` design-it-twice process. If the user rejects a candidate for a durable, surprising, trade-off-driven reason, offer an ADR so a future audit does not keep proposing it. Ephemeral timing or priority choices do not need an ADR.

The grilling loop is complete when no unresolved decision blocks a spec, every claimed structural improvement has a downstream verification method, runtime sensitivity is explicitly `none` or identified, every remaining empirical unknown has a concrete validation plan or downstream ticket, and the user confirms shared understanding. Record a **Structure Decision Brief**:

```text
Candidate and intended outcome:
Evidence and causal diagnosis:
Chosen owner and domain terms:
Chosen structural level and alternatives:
Why not one level smaller or larger:
Friction to remove:
Current and target tree:
Public interface and private roles:
Dependency direction:
Expected locality/interface/dependency/navigation/ownership/testability outcome:
Migration stages:
Verification contract for the downstream build:
Behavior-preservation checks:
Runtime-sensitive mechanisms touched: none / list
Conditional runtime validation:
Structural enforcement:
Affected tests, builds, exports, generated rules, docs, and ownership metadata:
Build/compatibility constraints:
Risks, rollback, and rejected alternatives:
Blocking open decisions: none
Validation questions and evidence plan:
Deferred decisions and revisit triggers:
```

Domain glossary or ADR edits may happen inline through `domain-modeling`; source-code restructuring does not.

### 4. Hand the decision into the build flow

Present the completed brief, carry its verification contract intact into the handoff, and recommend the next explicit user-invoked step. The downstream build must distinguish structural acceptance from behavior-preservation checks and require runtime validation only for identified runtime-sensitive mechanisms. This audit does not execute those checks. A user-invoked skill cannot invoke another user-invoked skill. In Codex, invite the user to run:

```text
$to-spec       turn the agreed decision into a durable spec
$to-tickets    divide a large migration into buildable slices with dependencies
$implement     execute the approved spec; it supplies TDD and code review
```

For clients whose explicit syntax is slash commands, use `/to-spec`, `/to-tickets`, and `/implement` instead. If Matt's project setup is absent, tell the user to invoke `setup-matt-pocock-skills` first. Explain that `to-spec` and `to-tickets` publish to the configured tracker, while `implement` may edit, review, commit, or otherwise perform the side effects defined by the installed upstream version. If the user wants no tracker publication, retain the brief and start a separate implementation request instead.

Use only the steps the scope needs. A small, already precise candidate may go from the brief directly to a separately requested implementation. The audit itself finishes before source changes begin.

## Completion

Finish with:

- the HTML report's absolute path;
- the selected candidate, report-only result, or no-change verdict;
- the confirmed Structure Decision Brief when grilling occurred;
- the downstream verification contract when grilling occurred;
- any domain documentation updated;
- the recommended next explicit skill invocation.
