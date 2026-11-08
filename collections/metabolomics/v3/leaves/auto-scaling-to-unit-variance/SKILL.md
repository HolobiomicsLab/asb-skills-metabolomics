---
name: auto-scaling-to-unit-variance
description: Use when a normalized metabolomics feature table (e.g. the MetaboReport
  cancerCell demo dataset) has mass features on incomparable scales and you need per-feature
  unit-variance scaling before statistical analysis.
when_to_use_negative:
- The feature table is already scaled/normalized to a common scale, so a second scaling
  pass would distort the data
- The downstream analysis requires preserving original intensity magnitudes (e.g.
  fold-change reporting on raw normalized values), which unit-variance scaling removes
- The goal is report generation or project documentation rather than preprocessing
  mass features for statistical comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: MetaboReport
  role: R Shiny app whose preprocessing module implements the six data scaling approaches,
    including auto scaling, on the normalized feature table
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment in which MetaboReport is installed as a package and scaling
    is applied to the feature table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:c46b21ac1a7ff4b3533570ec7f2db73543a7f34184f12efbc8df3abe71cc6bfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - dplyr
  - ropls
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# auto-scaling-to-unit-variance

## Summary

MetaboReport's auto scaling (unit-variance scaling) is one of six preprocessing scaling approaches — each mass feature's mean-centered values are divided by that feature's standard deviation so all features are on a comparable scale for downstream statistical analysis and visualization.

## When to use

Use when a normalized metabolomics feature table (e.g. the MetaboReport cancerCell demo dataset) has mass features on incomparable scales and you need per-feature unit-variance scaling before statistical analysis.

## When NOT to use

- The feature table is already scaled/normalized to a common scale, so a second scaling pass would distort the data
- The downstream analysis requires preserving original intensity magnitudes (e.g. fold-change reporting on raw normalized values), which unit-variance scaling removes
- The goal is report generation or project documentation rather than preprocessing mass features for statistical comparison

## Inputs

- Normalized metabolomics feature table (mass features x samples), e.g. the MetaboReport cancerCell demo dataset loaded into R

## Outputs

- Auto-scaled feature table with per-feature mean ~0 and SD = 1
- Six CSV files, one per scaling method (mean centering, auto, pareto, range, vast, level scaling)

## How to apply

Load the normalized feature table (MetaboReport cancerCell dataset) into R. First apply mean centering by subtracting each feature's column mean; then, for auto scaling specifically, divide those mean-centered values by each feature's standard deviation, giving every feature unit variance. This is one of six scaling methods MetaboReport provides — mean centering, auto scaling, pareto scaling (divide by sqrt of SD), range scaling (divide by value range), vast scaling (weight auto-scaled values by mean/SD ratio), and level scaling (divide by the mean) — applied during preprocessing so all mass features are aligned on a comparable scale. Auto scaling is appropriate when features should contribute equally regardless of absolute intensity; choose pareto or vast instead when partial variance preservation is desired. In MetaboReport, scaling is invoked through the app's preprocessing module (R/mod_*.R modules calling internal R/fct_*.R analysis functions), and each of the six scaled tables can be exported as CSV, one per method.

## Related tools

- **MetaboReport** (R Shiny app whose preprocessing module implements the six data scaling approaches, including auto scaling, on the normalized feature table) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment in which MetaboReport is installed as a package and scaling is applied to the feature table)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); MetaboReport::run_app()
```

## Evaluation signals

- After auto scaling, every feature's column mean is ~0 and its column standard deviation is ~1 (unit variance invariant)
- Confirm the centering step preceded scaling: auto scaling divides mean-centered values by the SD, not raw values
- Six scaled tables are produced and exported, one per method, with values matching the method formulas (pareto = mean-centered / sqrt(SD); vast = auto-scaled weighted by mean/SD ratio; level = value / mean)
- Scaled output retains the same feature and sample dimensions as the input normalized table

## Limitations

- Auto scaling equalizes variances but discards information about relative feature intensity magnitude; pareto or range scaling are alternatives when partial preservation is needed
- The README notes the supported public API is intentionally small (run_app()); scaling helpers are internal, so their implementation can evolve without a compatibility contract
- Version v0.3.6 fixed bugs in report generation and analysis speed, so behavior may vary across versions

## Evidence

- [methods] MetaboReport provides six data scaling approaches — mean centering, auto scaling, pareto scaling, range scaling, vast scaling, and level scaling — which are applied during preprocessing: [evidence span withheld — anchor status: not_found]
- [intro] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [abstract] simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting: "simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app.: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [methods] cancerCell (MetaboReport): "- `cancerCell (MetaboReport)`"
