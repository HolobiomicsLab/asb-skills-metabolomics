---
name: metabolomics-feature-scaling
description: Use when a normalized metabolomics feature table (mass features x samples)
  must be brought onto a comparable scale prior to statistical analysis or visualization,
  and a choice among centering/scaling variants is needed.
when_to_use_negative:
- Input is still a raw peak/feature table that has not been normalized — scaling should
  follow the preprocessing/normalization step
- Data are raw spectral files (e.g. mzML) rather than a tabular normalized feature
  matrix
- Only a single feature or a subset already on a common scale is being analyzed, where
  cross-feature scale alignment is unnecessary
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetaboReport
  role: Shiny app that performs the data preprocessing/scaling of metabolomics feature
    tables among its analysis modules
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Language in which the scaling transformations are computed and from which
    scaled tables are exported as CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:ffd64110cf05c59b7983b971fffedddfe16ca7611ca644f6f72d3fa99100fdf5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolomics-feature-scaling

## Summary

Applies the six data scaling methods provided by MetaboReport — mean centering, auto scaling, pareto scaling, range scaling, vast scaling, and level scaling — to a normalized metabolomics feature table so all mass features are aligned on a comparable scale before statistical analysis.

## When to use

Use when a normalized metabolomics feature table (mass features x samples) must be brought onto a comparable scale prior to statistical analysis or visualization, and a choice among centering/scaling variants is needed.

## When NOT to use

- Input is still a raw peak/feature table that has not been normalized — scaling should follow the preprocessing/normalization step
- Data are raw spectral files (e.g. mzML) rather than a tabular normalized feature matrix
- Only a single feature or a subset already on a common scale is being analyzed, where cross-feature scale alignment is unnecessary

## Inputs

- Normalized metabolomics feature table (mass features x samples), e.g. the MetaboReport cancerCell demo dataset
- Feature-wise means, standard deviations, and value ranges computed from the table columns

## Outputs

- Six scaled feature tables (one each for mean centering, auto, pareto, range, vast, and level scaling)
- CSV files, one per scaling method

## How to apply

Load the normalized demo feature table (e.g. the MetaboReport cancerCell dataset) into R. Apply mean centering by subtracting each feature's column mean; auto scaling by dividing mean-centered values by each feature's standard deviation; pareto scaling by dividing mean-centered values by the square root of each feature's standard deviation; range scaling by dividing each feature by its value range; vast scaling by weighting auto-scaled values with each feature's mean/SD ratio; and level scaling by dividing each feature by its mean. Choose the variant according to the downstream analysis (auto scaling for full variance equalization, pareto to retain partial signal magnitude, vast for low-intensity features, level/range for relative comparisons). Export each of the six resulting scaled tables as a separate CSV file, one per scaling method, so the effect of each transformation can be compared downstream.

## Related tools

- **MetaboReport** (Shiny app that performs the data preprocessing/scaling of metabolomics feature tables among its analysis modules) — https://github.com/YonghuiDong/MetaboReport
- **R** (Language in which the scaling transformations are computed and from which scaled tables are exported as CSV)

## Examples

```
library(MetaboReport); run_app()
```

## Evaluation signals

- Each scaled table's column means (or means/SDs) match the definition of the chosen scaling (e.g. auto-scaled columns have mean 0 and SD 1; level-scaled columns have mean 1)
- Six distinct output CSV files are produced, exactly one per scaling method, with identical row/column structure to the input feature table
- Pareto-scaled values equal mean-centered values divided by sqrt(SD), and vast-scaled values equal auto-scaled values multiplied by mean/SD, verifiable by recomputation
- No missing values or changed feature/sample identifiers are introduced relative to the normalized input table

## Limitations

- Scaling assumes the input table has already been normalized; applying it to unnormalized raw intensities mixes preprocessing stages
- In MetaboReport the analysis helpers are internal, so the scaling implementation is not part of the supported public API (only run_app() is exposed), meaning parameterization beyond the app's options is not contractually stable
- Choice of scaling variant is analysis-dependent; no single method is universally appropriate (e.g. auto scaling can over-amplify noise in low-intensity features, motivating pareto/vast alternatives)

## Evidence

- [other] six data scaling approaches: [evidence span withheld — anchor status: not_found]
- [abstract] data preprocessing scope: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] launching the app: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [methods] demo dataset: "- `cancerCell (MetaboReport)`"
