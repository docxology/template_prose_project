# `template_prose_project/docs/manuscript/`

Hand-curated manuscript and run policy for the prose-review exemplar.

## Sections

| File | Role |
|---|---|
| [`config.yaml`](config.yaml) | Single source of truth. All thresholds, paths, output toggles. |
| [`preamble.md`](preamble.md) | LaTeX preamble injected into Pandoc. |
| [`00_abstract.md`](00_abstract.md) | Abstract with `{{UPPER_NAME}}` substitution markers. |
| [`01_introduction.md`](01_introduction.md) | Motivation. |
| [`02_methodology.md`](02_methodology.md) | Five pipeline stages. |
| [`03_results.md`](03_results.md) | What the bundled run produces. |
| [`04_conclusion.md`](04_conclusion.md) | Summary + extensions. |
| [`05_pipeline_internals.md`](05_pipeline_internals.md) | Internal data structures (classDiagram). |
| [`06_reproducibility.md`](06_reproducibility.md) | Determinism guarantees. |
| [`99_references.md`](99_references.md) | Pandoc bridge to `references.bib`. |
| [`references.bib`](references.bib) | Hand-curated bibliography. **Read-only — pipeline validates, never writes.** |

## Editing

Register this checkout with the [pinned runner](../quickstart.md) before running
the review or rendering the manuscript.

```bash
# After editing manuscript files, re-run the review:
uv run python projects/working/template_prose_project/scripts/run_prose_pipeline.py
```

The pipeline:

1. Reads every `*.md` in this directory (excluding `preamble.md`).
2. Strips front-matter, fenced code, inline code, and link URLs.
3. Computes prose metrics, structure outline, quality flags.
4. Cross-checks every `[@key]` citation against `references.bib`.
5. Evaluates the configured threshold checks.
6. Writes `output/review_report.md` with the full breakdown.

See [AGENTS.md](AGENTS.md) for the substitution markers and editing checklist, and [SYNTAX.md](SYNTAX.md) for the project's Pandoc citation/cross-reference conventions.

## See also

- [`SYNTAX.md`](SYNTAX.md) — Pandoc citation/cross-reference syntax for this manuscript.
- [`../../../docs/guides/manuscript-semantics.md`](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/docs/guides/manuscript-semantics.md) — Repository-wide manuscript semantics.
- [`../../../AGENTS.md`](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/AGENTS.md#permanent-canonical-exemplars) — public exemplar roster.
