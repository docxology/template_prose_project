# Forking Guide — template_prose_project

> A 5-minute walkthrough for copying this exemplar into a new
> editorial-review project. The point of the guide is to make every
> decision explicit so a forker doesn't have to read every other doc to
> find out *what's required vs aesthetic, what's enforced vs convention,
> and what they'll hit friction on*.

## Start a fork

Create a fork of this standalone repository and register it under
`projects/working/my_review_project` in the [pinned runner](quickstart.md).
Replace the manuscript sections and bibliography in `docs/manuscript/`
and update the metadata and policy in `docs/manuscript/config.yaml`. Run
the tests and the prose review using the commands in the quickstart with
your project name.

## What you're forking

This template is a **prose-review pipeline**: it has **no algorithm of
its own** — instead, `scripts/run_prose_pipeline.py` calls
`infrastructure.prose.analyze_manuscript`, then `src/pipeline/` applies five
threshold checks (grade-level band, citation density, H1-per-file,
no-skipped-heading-levels, bibliography consistency via
`src/prose_facade.parse_bib_keys`), and emits JSON +
Markdown + three PNGs.

The transferable pattern is **self-grading**: the bundled
`docs/manuscript/config.yaml` is deliberately permissive (FKGL band 10–18,
citation floor 0.0) so the exemplar's own abstract passes its own gate.
A forker who tightens any threshold without rewriting the methodology
section will discover that the project's own documentation fails the
gate that the documentation describes. The
[`docs/manuscript/config.yaml.example`](manuscript/config.yaml.example)
ships a stricter starting point (FKGL 12–16, citation floor 3.0,
`fail_on_unused=true`) for a more realistic editorial workflow.

## REQUIRED vs AESTHETIC

A pipeline-enforced gate fails CI if you delete a REQUIRED path. An
AESTHETIC path is convention only. The full inventory lives in
[`AGENTS.md`](AGENTS.md); short version:

| Class | Examples | Action |
|---|---|---|
| REQUIRED — pipeline gate | All `src/*.py`, all `tests/test_*.py`, `tests/conftest.py` (pins `MPLBACKEND=Agg`), `pyproject.toml`, `docs/manuscript/config.yaml`, `docs/manuscript/*.md`, `docs/manuscript/references.bib`, `docs/manuscript/preamble.md` | Keep them; the 90% coverage gate + LaTeX render depend on them |
| REQUIRED — orchestration | `scripts/run_prose_pipeline.py`, `scripts/y_generate_prose_figures.py`, `scripts/z_generate_manuscript_variables.py` | Subprocess-tested in `tests/test_scripts.py`; the alphabetical prefix (none / `y_` / `z_`) is an ordering hint |
| AESTHETIC | `docs/*.md`, `*/STYLE.md`, `*/PATTERNS.md`, `*/CONVENTIONS.md`, `*/AGENTS.md`, `*/README.md`, `scripts/00_preflight.py`, `docs/manuscript/config.yaml.example` | Drift detected only by `scripts/audit/check_template_drift.py` and audits |

## Concrete first steps after fork

### 1. Drop in your manuscript
Replace `docs/manuscript/00_abstract.md` through `docs/manuscript/99_references.md`
with your own sections. Section files are CommonMark Markdown with
Pandoc `[@key]` citations. `docs/manuscript/references.bib` is
**hand-curated and read-only** — `_check_bibliography` validates that
every `[@key]` cited resolves to an entry but never writes to the bib.

### 2. Update thresholds
Every knob lives in `docs/manuscript/config.yaml`. The defaults are
permissive so the bundled prose passes; tighten them once your real
manuscript is in:

- `prose.target_grade_level_min` / `_max` — Flesch-Kincaid Grade Level band
- `prose.long_sentence_threshold` — words-per-sentence cutoff
- `prose.citation_density_min_per_1000` — minimum citations per 1000 words
- `bibliography.fail_on_missing` / `fail_on_unused` — strictness flags

The loader rejects unknown YAML keys and enforces invariants
(`target_grade_level_min < target_grade_level_max`, etc.) with
`ValueError` listing the offending value — see
[`faq.md`](faq.md#yaml-loader-rejects-my-new-key).

### 3. Add new checks
[`AGENTS.md`](../AGENTS.md) "Extending" documents the four-step process:

1. Add a field to `ProseAnalysisConfig` in `src/config.py` (and to the
   `_KNOWN_PROSE_KEYS` registry).
2. Add a `_check_<name>` function in `src/pipeline/checks.py`.
3. Wire it into `run_prose_pipeline` so it appears in
   `artifacts.checks`.
4. Add a test in `tests/test_pipeline.py` covering both
   `passed=True` and `passed=False`.

### 4. Run the drift checker before pushing
```bash
uv run python scripts/audit/check_template_drift.py
```
20 unit tests in
[`tests/infra_tests/test_check_template_drift.py`](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/tests/infra_tests/test_check_template_drift.py)
prove each detector catches the bug class it was built for.

## Common friction points (and fixes)

| Symptom | Cause | Fix |
|---|---|---|
| Tests collect 0 / coverage 0% | Per-project `.venv` lacks `pytest` (`uv venv` without `uv sync`) | The runner now hard-fails; the canonical gate is the `uv run pytest …` command above |
| `PDF Rendering` stage fails with `mmdc could not find Chrome` | `docs/manuscript/05_pipeline_internals.md` embeds `mermaid` blocks | One-time: `npx --yes puppeteer browsers install chrome-headless-shell`; `scripts/00_preflight.py` emits an actionable warning before the PDF stage |
| Matplotlib backend error in CI | `MPLBACKEND` not set | `tests/conftest.py` pins `MPLBACKEND=Agg` at import time — keep it |
| `Unknown prose key` ValueError | Typo in `docs/manuscript/config.yaml` (e.g., `target_grade_level_minimum`) | The strict loader rejects unknown keys; check spelling against `_KNOWN_PROSE_KEYS` in `src/config.py` |
| `{{TOKEN}}` appears literally in the rendered PDF | `scripts/z_generate_manuscript_variables.py` did not run or the token is not in `compute_variables()` | Re-run `z_generate_manuscript_variables.py`; if still literal, add to `ManuscriptVariables` |

## Sibling exemplar

[`template_code_project`](https://github.com/docxology/template_code_project) is the
numerical-research sibling — same shape, has its own gradient-descent
algorithm in `src/optimizer.py`. If your work is computational rather
than editorial, fork that one instead.

## See also

- [`AGENTS.md`](AGENTS.md) — full doc inventory and reading order
- [`agent_instructions.md`](agent_instructions.md) — 7 hard rules
- [`architecture.md`](architecture.md) — module boundaries
- [`testing_philosophy.md`](testing_philosophy.md) — zero-mock standard
- [`rendering_pipeline.md`](rendering_pipeline.md) — 4-phase pipeline
- [`faq.md`](faq.md) — recurring questions
- [`../../../scripts/audit/check_template_drift.py`](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/scripts/audit/check_template_drift.py) — the drift checker
