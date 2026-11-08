---
name: quantile-normalization-of-sample-distributions
description: Use when an imputed metabolomics feature table (samples × metabolite
  features) shows sample-to-sample intensity distribution shifts to remove before
  statistical analysis, and simple per-sample scaling is insufficient.
when_to_use_negative:
- Input is raw, not-yet-imputed intensities or a peak list — MetaboReport's normalization
  operates on the imputed feature table downstream of imputation.
- The intended correction is single-factor per-sample scaling (internal standard-based,
  linear baseline mean/median, or probabilistic quotient normalization) rather than
  forcing identical empirical distributions across samples.
- The table contains only one sample — there are no cross-sample distributions to
  align.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboReport
  role: R Shiny web app whose data preprocessing module implements quantile normalization
    (among five methods) on imputed feature tables; launched via run_app()
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime language for installing MetaboReport from GitHub with remotes and
    starting the app
- name: Shiny
  role: Framework hosting the preprocessing GUI in which quantile normalization is
    executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:309721848c8c2e4381b9006fa39c37b4e2b4b24d8db82855bf57288c56f7f73b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# quantile-normalization-of-sample-distributions

## Summary

Quantile normalization of metabolomics feature tables as implemented in MetaboReport's data preprocessing module: each sample's feature-intensity distribution is forced onto the same empirical distribution. This removes sample-to-sample variation before statistical analysis and reporting.

## When to use

Use when an imputed metabolomics feature table (samples × metabolite features) shows sample-to-sample intensity distribution shifts to remove before statistical analysis, and simple per-sample scaling is insufficient.

## When NOT to use

- Input is raw, not-yet-imputed intensities or a peak list — MetaboReport's normalization operates on the imputed feature table downstream of imputation.
- The intended correction is single-factor per-sample scaling (internal standard-based, linear baseline mean/median, or probabilistic quotient normalization) rather than forcing identical empirical distributions across samples.
- The table contains only one sample — there are no cross-sample distributions to align.

## Inputs

- Imputed metabolomics feature table: samples as rows, metabolite feature intensities as columns (e.g., the cancerCell demo dataset shipped with MetaboReport)

## Outputs

- Quantile-normalized feature table exported as CSV, preserving the exact samples and metabolite features of the imputed input table

## How to apply

Start from the imputed feature table (samples as rows, metabolite feature intensities as columns) — e.g., the cancerCell demo dataset shipped with MetaboReport — since normalization in this pipeline runs downstream of imputation. Launch the Shiny app with MetaboReport::run_app() and apply quantile normalization in the preprocessing module, so that the feature intensity distributions of all samples are forced to the same empirical distribution (mechanically: rank intensities within each sample, compute a shared value per rank across samples, and back-assign to original ranks). Choose quantile normalization over the sibling methods — internal standard-based, linear baseline on mean or median, and probabilistic quotient normalization — when distributional shape differences, not just overall per-sample intensity offsets, drive the sample-to-sample variation. In the reconstruction workflow, apply all five methods and export one CSV table per method. Validate that each output CSV preserves the dimensions of the imputed input table (same samples, same features). Note the package API is intentionally only run_app(); normalization helpers are internal (R/fct_*.R), so any parameters must be set via the GUI or verified by inspecting the source.

## Related tools

- **MetaboReport** (R Shiny web app whose data preprocessing module implements quantile normalization (among five methods) on imputed feature tables; launched via run_app()) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime language for installing MetaboReport from GitHub with remotes and starting the app)
- **Shiny** (Framework hosting the preprocessing GUI in which quantile normalization is executed)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); MetaboReport::run_app()
```

## Evaluation signals

- Each normalized CSV preserves the dimensions of the imputed input table — same samples and same metabolite features (the stated validation in the reconstruction workflow).
- After normalization, the sorted feature-intensity vectors are identical across samples — the defining invariant of forcing all samples to the same empirical distribution.
- Running the full normalization module yields five output tables (internal standard-based, linear baseline mean, linear baseline median, PQN, quantile), each with matching dimensions.
- Intensity values remain finite and non-negative, and no samples or features are dropped or reordered relative to the input.

## Limitations

- The supported public package API is intentionally only run_app(); normalization helpers are internal, so quantile normalization cannot be called as a standalone documented function and parameters must be inspected via the GUI or in the R/fct_*.R source.
- The article and README do not document quantile-normalization parameters (e.g., reference sample or rank-averaging scheme); the standalone desktop build is distributed via SourceForge rather than CRAN, so reproducibility depends on the GitHub commit.
- The README changelog notes bugs fixed in report generation in v0.3.6, so report outputs from earlier versions may be unreliable.

## Evidence

- [other] MetaboReport implements five normalization methods to reduce sample-to-sample variation: [evidence span withheld — anchor status: not_found]
- [other] quantile normalization forces identical empirical distributions: [evidence span withheld — anchor status: not_found]
- [intro] preprocessing scope of the app: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] public API limited to run_app(); helpers internal: "The supported public package API is intentionally small: run_app(). Analysis helpers are internal so their implementation can evolve without creating an accidental compatibility contract."
- [methods] starting the app: "you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app"
