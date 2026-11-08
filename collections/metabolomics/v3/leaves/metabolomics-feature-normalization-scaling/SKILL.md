---
name: metabolomics-feature-normalization-scaling
description: Use when a preprocessed metabolomics or proteomics feature table (after
  missing-value imputation) still contains features on incomparable numeric scales
  and the goal is normalized data for downstream EDA or statistical analysis in POMAShiny/POMA.
when_to_use_negative:
- Input feature table has not yet passed the preprocessing panel (missing-value filtering
  at the 20% default threshold and imputation) — the normalization panel is the second
  of three sequential steps.
- The analysis goal is spectral data processing or raw MS workflow operations rather
  than statistical normalization of an already-quantified feature matrix.
- Data already carry a documented prior transformation/scaling that must be preserved
  (e.g. normalized in an external pipeline) — re-normalizing would destroy comparability.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: POMAShiny
  role: web interface exposing the normalization panel that applies transformation-and-scaling
    in a single step to the imputed feature matrix
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end package providing the normalization functions invoked
    by the POMAShiny normalization panel
  repo: https://github.com/nutrimetabolomics/POMA
- name: MSnbase
  role: R package supplying the S4 container structure holding the target and features
    data that the normalization functions operate on
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:adf87af5705dc63953b16472e0d3472a87b3ff19b58c9025dbe8d924cce7d2a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - MSnbase
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# metabolomics-feature-normalization-scaling

## Summary

Transform and scale metabolomics/proteomics feature matrices (CSV features file loaded into an MSnbase S4 container via POMA/MSnbase) using POMAShiny's normalization panel, which combines data transformation and scaling in a single step with six available methods.

## When to use

Use when a preprocessed metabolomics or proteomics feature table (after missing-value imputation) still contains features on incomparable numeric scales and the goal is normalized data for downstream EDA or statistical analysis in POMAShiny/POMA.

## When NOT to use

- Input feature table has not yet passed the preprocessing panel (missing-value filtering at the 20% default threshold and imputation) — the normalization panel is the second of three sequential steps.
- The analysis goal is spectral data processing or raw MS workflow operations rather than statistical normalization of an already-quantified feature matrix.
- Data already carry a documented prior transformation/scaling that must be preserved (e.g. normalized in an external pipeline) — re-normalizing would destroy comparability.

## Inputs

- target (metadata) CSV file
- features CSV file (samples × features, post-imputation)
- MSnbase S4 container assembled via POMA
- selected normalization method (autoscaling, level scaling, log scaling, log transformation, vast scaling, or log pareto scaling)

## Outputs

- normalized feature table (single transformation-and-scaling step applied)
- normalized data carried forward to POMAShiny EDA and statistical analysis panels

## How to apply

Load the target (metadata) and features CSV files into an MSnbase S4 container using the POMA back-end, complete the upstream preprocessing panel steps (feature filtering at the 20% missing-value threshold per study group, imputation from the six offered methods including k-nearest neighbours), then open the normalization panel. Choose one of six methods — autoscaling, level scaling, log scaling, log transformation, vast scaling, or log pareto scaling — where each combines transformation and scaling in a single step rather than separate operations. Apply the chosen normalization to the imputed feature matrix, then export the normalized table for exploratory data analysis and downstream statistical panels; the output must be a complete numeric matrix with no missing values and features retained by the missingness filter.

## Related tools

- **POMAShiny** (web interface exposing the normalization panel that applies transformation-and-scaling in a single step to the imputed feature matrix) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end package providing the normalization functions invoked by the POMAShiny normalization panel) — https://github.com/nutrimetabolomics/POMA
- **MSnbase** (R package supplying the S4 container structure holding the target and features data that the normalization functions operate on)

## Examples

```
shiny::runApp(appDir = "app")
```

## Evaluation signals

- Output feature table contains no missing values after normalization (imputation already completed upstream).
- Only features that survived the 20% missing-value filter per study group are present in the normalized output.
- The selected method is one of the six documented options: autoscaling, level scaling, log scaling, log transformation, vast scaling, log pareto scaling.
- Transformation and scaling were applied as a single combined step, not as two separate sequential operations.
- Normalized matrix dimensions match the input (samples × retained features), with row/column order preserved for downstream EDA.

## Limitations

- POMAShiny requires two CSV input files (target/metadata and features); other formats are not accepted by the upload panel.
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer bs4Dash versions due to substantial breaking changes in the API.
- The back-end uses MSnbase (older README) or SummarizedExperiment (newer README); container class must match the installed POMA version.

## Evidence

- [results] the normalization process comprises both the transformation and scaling of the data in a single step: "the normalization process comprises both the transformation and scaling of the data in a single step"
- [results] The methods available for this purpose are autoscaling, level scaling, log scaling, log transformation, vast scaling, and log pareto scaling: "The methods available for this purpose are autoscaling, level scaling, log scaling, log transformation, vast scaling, and log pareto scaling"
- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] POMAShiny's back-end structure uses the POMA [13], MSnbase [14] and tidyverse family [15] R packages: [evidence span withheld — anchor status: not_found]
- [results] users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default): "users are able to remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
- [readme] POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API: "POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API"
