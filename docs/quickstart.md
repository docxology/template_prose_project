# Run the prose review and render the manuscript

The standalone checkout contains the editable sources and a published output snapshot. To build this checkout, register it with the shared runner. The manuscript starts at [`docs/manuscript/`](manuscript/README.md).

## Set up the pinned runner

Install Python 3.14+, uv, Node.js with npm, Pandoc, a version-compatible `pandoc-crossref` on `PATH`, and XeLaTeX with the [template's TeX packages](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/README.md#L565-L574). Mermaid also needs a Chrome-compatible browser. Set `PROSE_CHECKOUT` to the absolute path of a clean checkout of this repository at the commit to build. Run these commands from a directory where you can clone the template:

```sh
git clone https://github.com/docxology/template.git prose-template-runner
cd prose-template-runner
git checkout --detach 87704feae0a4490ef7dad9c931d268e8c30273c2
uv sync --locked
npm ci
export PATH="$PWD/node_modules/.bin:$PATH"
npx --no-install puppeteer browsers install chrome-headless-shell

project=projects/working/template_prose_project
mkdir -p projects/working
git -C "$PROSE_CHECKOUT" worktree add --detach "$PWD/$project" HEAD
# Use a fresh output tree so Stage 03 cannot consume the published snapshot.
mv "$project/output" "$(mktemp -d)/published-output"
```

The renderer can also use system Chromium. With Chromium installed, run `PUPPETEER_SKIP_DOWNLOAD=1 npm ci` and set `PUPPETEER_EXECUTABLE_PATH` to its executable path in place of the browser install above. The pinned Mermaid runner rejects JavaScript overrides. Run in a shell without `NODE_PATH` or `NODE_OPTIONS` set.

## Check sources and generate analysis

From the pinned template root with the working project registered:

```sh
uv run --no-sync --with pytest --with pytest-cov pytest "$project/tests" \
  --cov="$project/src" --cov-fail-under=90
uv run python "$project/scripts/run_prose_pipeline.py" --strict
uv run python "$project/scripts/y_generate_prose_figures.py"
uv run python "$project/scripts/z_generate_manuscript_variables.py"
```

The strict review reads [`config.yaml`](manuscript/config.yaml) and the [bibliography](manuscript/references.bib), then writes the review report and JSON checks under project-local `output/`. The next two scripts generate three diagnostic PNGs and hydrate the abstract variables in `output/manuscript/`. Edit `prose` and `bibliography` in `config.yaml` to tune the checks, and rerun all three scripts after changing the manuscript.

## Render and validate

```sh
uv run python scripts/pipeline/stage_03_render.py --project working/template_prose_project
uv run python scripts/pipeline/stage_04_validate.py --project working/template_prose_project
```

Open `$project/output/pdf/template_prose_project_combined.pdf` and `$project/output/web/index.html`. Check the hydrated abstract, numbered sections, bibliography, and the two Mermaid diagrams in Pipeline Internals. The three prose diagnostic plots are standalone files in `output/figures/`. Inspect `output/slides/` for the enabled per-section slide PDFs and `output/reports/validation_report.json` and `output/reports/rendered_provenance.json` for validation and build provenance. See the [output inventory](output_conventions.md) for each file's producer.

From the project checkout, run `git diff --check` and check Markdown navigation offline with Lychee:

```sh
lychee --offline README.md STANDALONE.md AGENTS.md 'docs/**/*.md'
```
