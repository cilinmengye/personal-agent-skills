# Structure Review HTML Report

Read this reference only when producing the candidate report in Phase 2.

## Output contract

Create one self-contained HTML file in the operating system's temp directory, named `structure-review-<YYYYMMDD-HHMMSS>.html`. Resolve the directory through the platform's temp-directory API or environment rather than assuming `/tmp`. Open the finished file for the user and report its absolute path. In a headless or remote environment, opening may be unavailable; still return the absolute path or a client-supported clickable file link.

The file contains inline CSS, inline SVG, and machine-readable JSON data only; it contains no executable JavaScript or external dependency. It must remain useful offline and must not write assets, scripts, or data into the repository. Match the report's prose language and `<html lang>` to the user's language while preserving code identifiers and paths. Escape all repository-derived text before inserting it into HTML.

The report is an inspection artifact, not an implementation plan. It presents evidence and candidate shapes; it does not silently settle public interfaces or edit source files.

## Page structure

1. **Header** — repository, inspected scope, current commit, history window or user-provided focus, date, and previous-report baseline when supplied.
2. **Audit verdict** — number of candidates by recommendation strength, plus a plain statement when no change is justified.
3. **Delta summary** — when a previous report is supplied, show new, changed, carried-forward, resolved, and durably rejected candidates.
4. **Candidate cards** — ordered by expected structural payoff and confidence, not by file size.
5. **Top recommendation** — one candidate and why it is the best next discussion, or `No structural change recommended`.
6. **Inspection appendix** — areas inspected, important evidence sources, resolved items, and rejected false positives.

Give every candidate a stable ID such as `S-01` so the user can select it unambiguously in the next turn. Preserve the ID of a matching candidate from the previous report and never recycle an ID within one report lineage; allocate the next unused number only for a new candidate.

## Delta contract

When a previous report is supplied, label every prior or current candidate as one of:

- `new` — absent from the previous report;
- `changed` — same structural concern, but evidence, scope, or target shape changed;
- `carried-forward` — materially unchanged and still relevant;
- `resolved` — evidence shows the friction is gone;
- `rejected` — the user recorded a durable reason not to pursue it.

Embed a machine-readable summary in `<script type="application/json" id="structure-review-data">`. Include report lineage, baseline, stable ID, status, strength, timing (`Current` or `Prospective`), chosen structural level, involved paths, the success-contract summary, and a concise fingerprint based on the concern plus owner. Encode `<` as `\u003c` inside the JSON so repository text cannot close the script element. This is data, not executable application code.

## Candidate card

Each card contains:

- **Title and badges:** stable ID, delta status, `Current` or `Prospective`, move type, and `Strong`, `Worth exploring`, or `Speculative`.
- **Why now:** the user goal, active hotspot, repeated conflict, or other evidence that makes this timely.
- **Files and modules:** concrete paths and the owning module in domain language.
- **Observed friction:** facts separated from inference, with file plus symbol/line or commit references where available.
- **Structural diagnosis:** how ownership, locality, interface, or dependency direction creates the friction.
- **Structural level:** the smallest sufficient move, why it resolves the friction, why one adjacent level smaller is insufficient, and why one adjacent level larger is excessive.
- **Before / possible after:** a side-by-side tree or dependency view using real paths. Label the right side `possible`, not `final`.
- **Public and private shape:** provisional interface hypothesis, private implementation area, and allowed dependency direction.
- **Structural success hypothesis:** the specific locality, interface, dependency, navigation, ownership, or testability improvement and how a downstream build can verify it. A cleaner-looking tree alone is not a benefit.
- **Behavior-preservation guardrails:** existing behavior, import, export, build, registry, or compatibility checks that protect the move without being presented as evidence of structural improvement.
- **Runtime sensitivity:** `none`, or the import/loading, dynamic-dispatch, registry/plugin, process/IPC, GPU-hot-path, synchronization, or package/build mechanism that makes conditional runtime validation necessary.
- **Change surface:** tests, manifests, exports, generated rules, documentation, and ownership metadata likely affected.
- **Risks and counterevidence:** what could make the move worse or unnecessary.
- **Open decisions:** questions reserved for the grilling phase.

Use prose only where it carries evidence or a decision. Visuals must represent observed paths or dependencies, not decorative boxes.

## Recommended visuals

Choose the smallest visual that proves the point:

- **Tree diff:** current and possible directory trees for placement, split, merge, or relocation candidates.
- **Dependency arrows:** current leakage or reverse imports versus the intended direction.
- **Public/private cutaway:** one public entry point with private implementation files behind it.
- **Change cluster:** files repeatedly changing together versus an ownership-based grouping.
- **Build/runtime map:** only when the candidate crosses a real operational boundary.

Use `<pre>` for trees and inline SVG for arrows or simple graphs. Include text labels so the report remains understandable without color.

## Minimal scaffold

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Structure review — {{repo}}</title>
  <style>
    :root { color-scheme: light; --ink:#172033; --muted:#64748b; --line:#dbe2ea;
      --paper:#f7f7f4; --card:#fff; --strong:#087f5b; --explore:#a16207;
      --spec:#64748b; --risk:#b42318; --accent:#334eac; }
    * { box-sizing:border-box; }
    body { margin:0; background:var(--paper); color:var(--ink);
      font:15px/1.55 ui-sans-serif,system-ui,-apple-system,"Segoe UI",sans-serif; }
    main { width:min(1120px,calc(100% - 32px)); margin:0 auto; padding:48px 0 72px; }
    header,.summary,.candidate,.appendix { background:var(--card); border:1px solid var(--line);
      border-radius:16px; padding:24px; margin-bottom:20px; }
    h1,h2,h3,p { margin-top:0; } h1 { font-size:30px; } h2 { font-size:22px; }
    .meta,.muted { color:var(--muted); } .badges { display:flex; gap:8px; flex-wrap:wrap; }
    .badge { border:1px solid currentColor; border-radius:999px; padding:3px 9px;
      font-size:12px; font-weight:700; }
    .strong { color:var(--strong); } .explore { color:var(--explore); }
    .speculative { color:var(--spec); }
    .grid { display:grid; grid-template-columns:1fr 1fr; gap:16px; }
    .panel { border:1px solid var(--line); border-radius:12px; padding:16px; min-width:0; }
    pre { overflow:auto; background:#f8fafc; border-radius:8px; padding:14px; }
    code,pre { font-family:ui-monospace,SFMono-Regular,Consolas,monospace; }
    .facts { border-left:4px solid var(--accent); padding-left:12px; }
    .risks { border-left:4px solid var(--risk); padding-left:12px; }
    @media (max-width:760px) { .grid { grid-template-columns:1fr; } }
    @media print { body { background:white; } main { width:100%; padding:0; }
      .candidate,.summary,header,.appendix { break-inside:avoid; box-shadow:none; } }
  </style>
</head>
<body>
  <main>
    <header>
      <h1>Structure review — {{repo}}</h1>
      <div class="meta">{{scope · commit · date · baseline}}</div>
    </header>
    <section class="summary">{{audit verdict and strength counts}}</section>
    <section aria-label="Candidates">
      <article class="candidate" id="S-01">
        <div class="badges">{{ID · move type · strength}}</div>
        <h2>{{candidate title}}</h2>
        <p class="facts">{{observed evidence}}</p>
        <div class="grid">
          <section class="panel"><h3>Current</h3><pre>{{current tree}}</pre></section>
          <section class="panel"><h3>Possible</h3><pre>{{possible tree}}</pre></section>
        </div>
        <section><h3>Diagnosis</h3>{{structural diagnosis}}</section>
        <section><h3>Structural level</h3>{{chosen level · why sufficient · why not smaller/larger}}</section>
        <section><h3>Provisional public/private shape</h3>{{interface hypothesis and dependency direction}}</section>
        <section><h3>Structural success hypothesis</h3>{{verifiable locality · interface · dependency · navigation · ownership · testability outcome}}</section>
        <section><h3>Behavior-preservation guardrails</h3>{{behavior · imports · exports · build · registry · compatibility}}</section>
        <section><h3>Runtime sensitivity</h3>{{none or mechanism requiring conditional validation}}</section>
        <section><h3>Change surface</h3>{{tests · manifests · exports · generated rules · owners}}</section>
        <section class="risks"><h3>Risks and counterevidence</h3>{{risks}}</section>
        <section><h3>Open decisions</h3>{{questions reserved for grilling}}</section>
      </article>
      {{more candidate cards}}
    </section>
    <section class="summary">{{top recommendation or no-change verdict}}</section>
    <section class="appendix">{{inspection scope and rejected false positives}}</section>
  </main>
  <script type="application/json" id="structure-review-data">{{escaped review JSON}}</script>
</body>
</html>
```

## Quality checks

- Every current path and relationship is traceable to inspected repository evidence; possible paths may come from an explicit planned change and remain provisional.
- Prospective candidates are traceable to both an explicit planned change and the repository's current seams.
- Facts and architectural inference are visibly distinct.
- Every candidate chooses the smallest sufficient structural level and explains why an adjacent smaller and larger level is wrong.
- The possible tree preserves a small public interface and gives private files credible owners.
- Every candidate states at least one material structural improvement and how a downstream build can verify it; directory neatness and passing behavior tests are not structural benefits by themselves.
- Behavior-preservation guardrails are distinct from structural acceptance.
- Runtime validation appears only when an identified runtime-sensitive mechanism changes; otherwise the report states `none`.
- Recommendation strength reflects evidence and migration risk.
- The report includes counterevidence and can conclude that no structural change is justified.
- Delta reports preserve candidate IDs and account for every candidate from the previous report.
- The file opens without network access and remains readable on a narrow screen and when printed.
