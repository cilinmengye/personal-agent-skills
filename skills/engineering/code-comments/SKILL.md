---
name: code-comments
description: Improve contracts, rationale, and conditional visual structure in human-maintained Python, C++, and CUDA systems code.
disable-model-invocation: true
license: MIT
---

# Code Comments

Improve code-adjacent documentation and scanability in human-maintained Python, C++, and CUDA systems code. Add information a caller or maintainer cannot reliably derive from names, types, and control flow; preserve useful visual structure without imposing one personal layout on every repository.

This skill is explicitly invoked. It does not choose the implementation, redesign modules, or automatically invoke another skill.

Settled behavior, public interfaces, and implementation choices take precedence. When this skill is invoked alone, surface any naming, type, control-flow, interface, or module-boundary change needed for clarity instead of making that implementation change under a comment-only scope. When an implementation skill is also explicitly active, let it own the code change, then document only the residual knowledge the code still cannot express.

## Establish local authority

Before editing, inspect the repository's formatter, lint rules, docstring or documentation format, comment tokens, nearby public APIs, and representative files. Repository rules take precedence, followed by the language formatter and official language conventions. Use the defaults below only where the project is silent.

The inspection is complete when the applicable local format and any public-contract convention are known, or their absence is explicit.

## Make the code carry what it can

Assess first whether a precise name, type, control-flow shape, assertion, or narrow interface could express the fact directly, subject to the scope boundary above. A comment should preserve information that would otherwise require reconstruction: purpose, rationale, invariants, units, state transitions, ordering, side effects, ownership, failure behavior, or a non-obvious algorithm.

Comments may explain **how** when a complex algorithm cannot be made self-evident without losing locality. Keep the explanation next to the code whose maintenance depends on it.

## Document contracts by audience

### Public interfaces

Document obligations and outcomes that a caller cannot infer from the signature:

- accepted values, units, shapes, and identity or ordering semantics;
- ownership, lifetime, mutation, aliasing, and thread/async safety;
- side effects, errors, partial success, cleanup, and required call order;
- performance assumptions that are part of the supported contract.

For Python, write informative docstrings for public modules, exported functions and classes, and public methods. Omit sections that merely repeat names, type annotations, or the function name. Follow the repository's docstring convention.

For C++ and CUDA, use the repository's documentation form for public headers, kernel wrappers, and cross-module seams.

### Private implementation

Add a private docstring or comment only when a maintainer still needs non-obvious contract, unit, state, invariant, error, ordering, side-effect, or rationale information after reading the code. A private helper whose name, types, and body already tell the whole story needs no template.

## Use semantic markers conditionally

Prefer the repository's recognized tokens. Where none exist, use this small fallback set:

- `WARNING:` for correctness or safety constraints whose violation causes serious failure;
- `PERF:` for measured or contractually important performance assumptions;
- `NOTE:` for non-obvious rationale, invariants, or API quirks;
- `TODO:` for a concrete incomplete condition or deferred improvement;
- `FIXME:` for a known defect with a concrete failure condition.

For `TODO` and `FIXME`, state what remains and when it is considered done; attach the repository's issue identifier when a tracker exists. Do not replace a project's established searchable tokens with this fallback vocabulary.

## Preserve conditional visual structure

Exact blank-line counts belong to the repository formatter and language convention. In Python, follow the project's PEP 8-compatible formatting. Where the project is silent, use two blank lines around top-level functions and classes, and one blank line between method definitions inside a class. Do not hand-format against the formatter.

Inside a function, a blank line may mark a real phase transition such as validation → mutation, setup → execution, or execution → cleanup. Keep tightly related statements together; a slight responsibility change does not automatically earn another gap.

A multi-statement branch receives a leading comment only when its condition and body fail to reveal a business intention, invariant, concurrency constraint, or ordering reason. An obvious branch remains uncommented.

Use a zone anchor only when all of these are true:

- the marker is at file top level, not inside a function;
- the file contains several stable, independently nameable regions;
- keeping those regions together improves locality more than splitting the file;
- the anchor does not conceal a multi-responsibility file that should be restructured.

Prefer the repository's section-marker form. Where the repository is silent, this is the fallback:

```python
# ===========================
# Scheduling Policy
# ===========================
```

The formatter or nearby convention decides surrounding blank lines.

## CUDA contracts

When editing a public kernel, public wrapper, or cross-module CUDA seam whose types do not express its memory and execution contract, read [references/CUDA-CONTRACTS.md](references/CUDA-CONTRACTS.md) and apply the relevant fields. Scalar parameters describe units, range, and meaning; they are not memory locations and do not receive a `[Host]` label.

## Completion check

Finish when every added or changed comment contributes information, public contracts expose caller obligations, visual grouping follows local conventions and real phases, and any code-level clarity problem outside this skill's scope is reported to the owning implementation or planning workflow rather than hidden by more commentary.
