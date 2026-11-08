---
name: metabolomics-intensity-transformation-and-scaling
description: Use when raw targeted or untargeted feature tables contain missing intensity
  values and non-normal, heteroscedastic intensities that must be transformed before
  running t-tests, fold-change, ANOVA, or regression in GetFeatistics.
when_to_use_negative:
- Input is already a processed feature table that has been filtered against pooled
  QC criteria and normalized — re-applying transf_data would double-transform intensities.
- Raw vendor MS files that still need feature extraction with XCMS or patRoon — this
  skill operates on extracted feature tables, not chromatograms.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: GetFeatistics
  role: Provides the transf_data function that performs missing-value replacement,
    log-transformation, and pareto scaling of feature intensities.
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime environment in which GetFeatistics is installed and the transformation
    workflow is executed.
- name: patRoon
  role: Upstream tool whose imported feature tables can serve as input to the intensity
    transformation step for untargeted studies.
  repo: rickhelmus/patRoon
- name: MS-DIAL
  role: Alternative upstream feature-table generator whose imported tables can be
    transformed with transf_data.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:db368197773c537d4b30aca982e77e36bfe1fa20eeb16e38cdd193e8b1abadfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metabolomics-intensity-transformation-and-scaling

## Summary

Prepares metabolomics intensity matrices for univariate statistics by replacing missing values, log-transforming raw feature intensities, and applying pareto scaling, as implemented in the GetFeatistics transf_data function.

## When to use

Use when raw targeted or untargeted feature tables contain missing intensity values and non-normal, heteroscedastic intensities that must be transformed before running t-tests, fold-change, ANOVA, or regression in GetFeatistics.

## When NOT to use

- Input is already a processed feature table that has been filtered against pooled QC criteria and normalized — re-applying transf_data would double-transform intensities.
- Raw vendor MS files that still need feature extraction with XCMS or patRoon — this skill operates on extracted feature tables, not chromatograms.

## Inputs

- Targeted or untargeted metabolomics feature table (feature × sample intensity matrix)
- Sample metadata / descriptor table (e.g. df_example_sample_data) defining sample groups for QC-aware filtering and downstream grouping

## Outputs

- Transformed intensity matrix with missing values replaced, log-transformed intensities, and pareto scaling applied
- Input matrix suitable for downstream GetFeatistics statistics functions (t-test, fold change, ANOVA, regression) and descriptive tables

## How to apply

Load the feature table (targeted or untargeted, the latter importable from patRoon or MS-DIAL) together with the sample metadata into R. Apply GetFeatistics' transf_data, which first replaces missing intensity values (pooled-QC-aware imputation is part of the package's pre-processing), then log-transforms the intensities to address non-normality, and finally applies pareto scaling so that features contribute comparably to the analysis without inflating noise-dominated variables. The transformed matrix is then passed to gentab_descr, gentab_P.t.test, gentab_FC, gentab_P.1wayANOVA_posthocTukeyHSD, gentab_P.2wayANOVA_posthocTukeyHSD, or the mixed-effect/Tobit regression functions. The rationale is that these univariate and regression models assume approximately normal, comparable-scale intensities; raw LC-MS intensities violate both assumptions.

## Related tools

- **GetFeatistics** (Provides the transf_data function that performs missing-value replacement, log-transformation, and pareto scaling of feature intensities.) — github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime environment in which GetFeatistics is installed and the transformation workflow is executed.)
- **patRoon** (Upstream tool whose imported feature tables can serve as input to the intensity transformation step for untargeted studies.) — rickhelmus/patRoon
- **MS-DIAL** (Alternative upstream feature-table generator whose imported tables can be transformed with transf_data.)

## Examples

```
library(GetFeatistics); transformed <- transf_data(feature_table, sample_metadata)
```

## Evaluation signals

- After transf_data, no missing values remain in the transformed intensity matrix (imputation completed).
- Transformed intensities are on a log scale and pareto-scaled — e.g. the mean of each feature's transformed values should be centered near zero with unit-scaled standard deviation (pareto convention).
- Downstream gentab_descr output on the transformed matrix should reflect the log/pareto-scaled distribution rather than the raw intensities.
- Running transf_data twice should be flagged as an error state: re-transformation of already-scaled values is not a valid workflow.

## Limitations

- Pooled-QC-based filtering of features must precede or accompany transformation — the article notes QC-filtering is a prerequisite pre-processing step before intensity transformation.
- The vignette demonstrates the workflow on mock data; the exact imputation value/strategy used by transf_data for missing values is not spelled out numerically in the article text.

## Evidence

- [intro] Data intensities are pre-processed to deal with missing values and data normality: "data intensities are pre-processed to deal with missing values and data normality"
- [full_text] Vignette workflow for statistical analysis on the mock data: [evidence span withheld — anchor status: not_found]
- [abstract] Import of feature tables from upstream tools: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL"
- [readme] README installation and workflow pointer: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing"
