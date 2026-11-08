---
name: statistical-outlier-detection
description: Use when a metabolomics or proteomics dataset has been loaded into POMAShiny
  (target + features CSV), missing values imputed and data normalized, and samples
  must be screened for statistical outliers before statistical analysis.
when_to_use_negative:
- Input is a raw, unprocessed feature table that still contains missing values — the
  outlier panel operates on the normalized output of the imputation and normalization
  steps, not on raw uploaded data.
- The goal is feature-level filtering (e.g. removing metabolites with >20% missing
  values per study group) rather than sample-level outlier removal — that is handled
  by the earlier missing-value filter step in the same panel.
- The analysis target is proteomics spectral data processing (e.g. peak picking, alignment)
  — POMAShiny's outlier step is a statistical post-processing operation, and the paper
  directs spectral work to the RforMassSpectrometry complementary packages.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: POMAShiny
  role: Web-based interface that hosts the outlier-detection and data-cleaning step
    as the final operation of its pre-processing panel, with customizable parameters
    for flagging and removing statistical outliers.
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end that powers POMAShiny's pre-processing operations,
    including outlier detection, enabling the same cleaning to be reproduced outside
    the web environment.
  repo: https://github.com/nutrimetabolomics/POMA
- name: MSnbase
  role: Back-end R package that provides the S4 container structure in which the feature
    data are assembled and on which the outlier-detection operation is executed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:cc604a44e00fcb7acda2620edbed0aa068db751b3b7329b554b356beb7fc9cf0
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# statistical-outlier-detection

## Summary

Detect and remove statistical outliers from pre-processed metabolomics or proteomics feature tables using POMAShiny's customizable outlier-detection and data-cleaning panel, which is the final step of the pre-processing workflow before exploratory data analysis.

## When to use

Use when a metabolomics or proteomics dataset has been loaded into POMAShiny (target + features CSV), missing values imputed and data normalized, and samples must be screened for statistical outliers before statistical analysis.

## When NOT to use

- Input is a raw, unprocessed feature table that still contains missing values — the outlier panel operates on the normalized output of the imputation and normalization steps, not on raw uploaded data.
- The goal is feature-level filtering (e.g. removing metabolites with >20% missing values per study group) rather than sample-level outlier removal — that is handled by the earlier missing-value filter step in the same panel.
- The analysis target is proteomics spectral data processing (e.g. peak picking, alignment) — POMAShiny's outlier step is a statistical post-processing operation, and the paper directs spectral work to the RforMassSpectrometry complementary packages.

## Inputs

- Normalized and imputed metabolomics or proteomics feature table held in a POMA/MSnbase S4 container built from a target (metadata) CSV and a features CSV
- Customizable outlier-detection parameters selected by the user in the POMAShiny pre-processing panel

## Outputs

- Set of samples flagged as statistical outliers
- Cleaned feature table with outlier samples optionally removed, ready for the exploratory-data-analysis and statistical-analysis panels

## How to apply

After completing the pre-processing panels (feature removal at the default 20% missing-values-per-study-group threshold, imputation, and normalization), navigate to the outlier-detection and data-cleaning step, which is the last operation in POMAShiny's pre-processing panel. The panel uses different customizable parameters to flag statistical outliers and offers the possibility of removing those samples from the analysis; the flagged samples are visualized interactively so the analyst can inspect which specimens are identified as outliers. Rationale: the paper recommends performing EDA and outlier screening before any statistical analysis because outliers can distort downstream univariate, multivariate, and machine-learning results. Because the step operates on the normalized, imputed POMA S4 container assembled from the uploaded target and features CSV files, it should be applied only after those two prior pre-processing steps have produced a complete (no missing values) and scaled feature table.

## Related tools

- **POMAShiny** (Web-based interface that hosts the outlier-detection and data-cleaning step as the final operation of its pre-processing panel, with customizable parameters for flagging and removing statistical outliers.) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end that powers POMAShiny's pre-processing operations, including outlier detection, enabling the same cleaning to be reproduced outside the web environment.) — https://github.com/nutrimetabolomics/POMA
- **MSnbase** (Back-end R package that provides the S4 container structure in which the feature data are assembled and on which the outlier-detection operation is executed.)

## Examples

```
docker run -d --rm -p 3838:3838 pomashiny; then open http://0.0.0.0:3838 and use the outlier-detection panel on the normalized feature table
```

## Evaluation signals

- The outlier step must be run on a feature table that contains no missing values — verify that the 20%-per-group missingness filter and an imputation method (e.g. k-nearest neighbours) have already been applied.
- The output must be a cleaned S4 object (or exported table) whose sample set equals the input sample set minus the flagged outliers; record which samples were removed.
- The flagged outlier list should be reported before any EDA or statistical-analysis panel is run, consistent with the paper's recommendation to perform EDA before statistical analysis.
- Cross-check the removal result between the POMAShiny interface and the POMA R package on the same pre-processed container — the paper states that the joint existence of both increases reproducibility of the analysis.

## Limitations

- Outlier-detection parameters are customizable but the article does not enumerate the specific statistical tests or thresholds used, so reproducibility of the exact cutoff requires consulting the POMA package source.
- The step operates on already-normalized data; applying it to raw or partially processed data will give misleading outlier calls.
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions of bs4Dash due to substantial breaking changes in the API, which can break local deployment of the outlier panel.
- No benchmarking or quantitative evaluation of POMAShiny's outlier detection against other tools is reported in the article.

## Evidence

- [results] The last step provided in this panel is outlier detection and data cleaning: "The last step provided in this panel is outlier detection and data cleaning"
- [results] removing statistical outliers using different customizable parameters: "the possibility of removing statistical outliers from the analysis using different customizable parameters"
- [results] it is highly recommended to perform an EDA before any statistical analysis: "it is highly recommended to perform an EDA before any statistical analysis"
- [intro] POMAShiny's back-end structure uses the POMA, MSnbase and tidyverse family R packages: [evidence span withheld — anchor status: not_found]
- [readme] POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions of bs4Dash: "NOTE: *POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API."
