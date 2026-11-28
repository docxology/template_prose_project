# Testing Philosophy: The Zero-Mock Standard

The Generalized Research Template strictly forbids mocking in
scientific/editorial validation. The prose-project test suite exemplifies
the standard: every check, every figure, every variable, and every script
is exercised against real Markdown, real BibTeX, real `tmp_path`
directories, and real subprocess invocation.

## Why Zero Mocks?

The prose project owns no algorithm of its own. Every line of `src/` is
either pure (config parsing, figure rendering, variable derivation, report
assembly, check evaluation) or a thin shell over `infrastructure/prose/`
(which is itself pure), with the `infrastructure` calls confined to the
`scripts/` layer. There is
nothing legitimate to mock: tests can always pass real text and assert on
real outputs.

If you find yourself wanting to mock something, treat it as a signal: the
work probably belongs in `infrastructure/`, not in `src/`. Move it and
test the boundary directly.

## The Validation Suite

Files in the standalone test suite. [Test directory](../tests/)

- `test_config.py` — covers `src/config.py` typed YAML loader (23 tests).
- `test_figures.py` — covers `src/figures.py` matplotlib renderers (6 tests).
- `test_manuscript_variables.py` — covers `src/manuscript_variables.py`
  substitution (11 tests).
- `test_pipeline.py` — covers `src/pipeline/` checks and
  `run_prose_pipeline` (44 tests across `TestRunProsePipeline`,
  `TestOptionalChecks`, `TestCheckUnits` (covers every `_check_<name>` in
  isolation — `_check_grade_level`, `_check_citation_density`,
  `_check_no_skipped_levels`, `_check_h1_per_file`, `_check_bibliography`),
  `TestCitationExtractionViaPipeline`, `TestProseRunArtifacts`,
  `TestCheckResult`, `TestLongSentenceThresholdWired`,
  `TestNegativeControls`).
- `test_pipeline_integration.py` — runs the bundled `docs/manuscript/` end-to-end
  against `run_prose_pipeline` (1 test).
- `test_prose_facade.py` — covers `src/prose_facade.py` report Protocols,
  `render_outline`, and `parse_bib_keys` (16 tests).
- `test_report.py` — covers `src/report.py::write_review_report`
  (15 tests).
- `test_scripts.py` — invokes the three orchestrator scripts via
  `subprocess.run` (4 tests).
- `test_claim_ledger.py` — binds `data/claim_ledger.yaml` sources to real
  files and re-checks each value against live code or manuscript prose.
- `test_llm_review.py` — offline, fail-closed coverage for the optional LLM
  review receipt and transcript validation.
- `test_manuscript_structure.py` — section labels are unique and every
  `[@sec:…]` cross-reference resolves to a declared `{#sec:…}` label.
- `test_report_schema.py` — strict `evidence_summary` schema regression
  controls (`src/pipeline/report_schema.validate_evidence_summary`).

The listed counts are a snapshot. For current standalone collection, use the
Quickstart's pytest command with `--collect-only -q` in place of the coverage
flags. Template's generated counts describe the bundled exemplars.
[Standalone test command](quickstart.md#check-sources-and-generate-analysis) ·
[Bundled counts](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/docs/_generated/COUNTS.md)

The test configuration sets `MPLBACKEND=Agg`, adds `src/` to `sys.path`, and
requires 90% branch coverage. [Conftest](../tests/conftest.py) ·
[Project configuration](../pyproject.toml)

## Coverage

Re-measure coverage with the standalone test command from the pinned runner
root. Add `--cov-report=term-missing` to see uncovered lines.
[Coverage command](quickstart.md#check-sources-and-generate-analysis)

Do not consume the buffer unnecessarily. Do not delete tests to make a
coverage number work — fix the gap.

## Real Inputs, Real Outputs

Every test uses real artefacts:

- **Real Markdown.** Tests write `.md` files to `tmp_path` and pass them
  through `infrastructure.prose.analyze_manuscript`. No string-faking, no
  pre-built `ManuscriptReport` objects in production-path tests.
- **Real BibTeX.** Tests covering `_check_bibliography` (emits
  `CheckResult(name="bibliography_consistency")`) write small but valid
  `.bib` files and `_check_bibliography` parses them through
  `src/prose_facade.parse_bib_keys`. This catches
  parser-level breakage that a mocked parse would hide.
- **Real `tmp_path`.** No test writes into the project's own `output/`
  tree; every filesystem operation is contained in pytest's
  `tmp_path` fixture.
- **Real subprocess.** `tests/test_scripts.py` invokes
  `scripts/run_prose_pipeline.py` and `scripts/y_generate_prose_figures.py`
  via `subprocess.run`, with real argparse, real exit codes, and real
  output-file existence checks.

## Integration Test

`tests/test_pipeline_integration.py::test_bundled_manuscript_runs` is the
end-to-end fixture: it copies the project's own `docs/manuscript/` directory
into a temporary location, runs `run_prose_pipeline` against it, and
verifies that `manuscript_report.json`, `checks.json`, `review_report.md`,
and `run_summary.json` all land in the expected locations. The assertion
target is structural — file existence and shape — rather than exact value,
so the test is stable across editorial revisions of the bundled manuscript.

## Zero-Mock Checklist

Before submitting any test, verify all boxes are checked:

- [ ] Test uses real Markdown strings or `.md` files written to `tmp_path`.
- [ ] Test calls `src/` functions with real `ManuscriptReport`,
  `ProjectConfig`, or `CheckResult` objects produced by infrastructure
  or by `src/config.py::load_project_config`.
- [ ] Test asserts on properties (passed/failed checks, file existence,
  field values), not on call counts.
- [ ] No `unittest.mock`, `MagicMock`, `create_autospec`, `@patch`, or
  `monkeypatch` used as a mock factory.
- [ ] Subprocess tests assert on exit codes and on the contents of files
  written by the script.

## Structural Rule: If You Need a Mock, Move the Code

The zero-mock constraint is self-enforcing when the architecture is correct:

- **`src/config.py`, `src/manuscript_variables.py`, `src/figures.py`,
  `src/report.py`, `src/prose_facade.py`** — pure modules → testable with
  real data.
- **`src/pipeline/`** — evaluates the configured checks over the
  pre-analysed report (pure; no `infrastructure` imports; bibliography
  cross-check via `src/prose_facade.parse_bib_keys`) → testable with real
  Markdown and real BibTeX in `tmp_path`.
- **`scripts/*.py`** — CLI shims that call `infrastructure.prose` /
  `infrastructure.rendering` → tested via `subprocess.run` in
  `tests/test_scripts.py`.

If you find yourself wanting to mock `analyze_manuscript`, `parse_bib_keys`,
or any I/O inside a test, stop. Either pass it real data, or move the
work behind a configuration knob and test both branches with real inputs.

## Running the Gate: Collection and Threshold, Not Just Exit Code

1. Confirm that pytest collected at least one test and reached 90% coverage.
   If collection is empty, check that the pinned runner has pytest available.
2. Run the direct command from the pinned runner root. Its `$project/tests`
   and `--cov="$project/src"` arguments select the same registered checkout.
   [Standalone test command](quickstart.md#check-sources-and-generate-analysis)

See [`troubleshooting.md`](troubleshooting.md#tests-report-passed-but-ran-0-tests--00-coverage).
