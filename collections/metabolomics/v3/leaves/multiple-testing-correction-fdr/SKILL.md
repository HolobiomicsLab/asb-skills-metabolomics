---
name: multiple-testing-correction-fdr
description: Use when a metabolomics or proteomics feature table (CSV) has been tested
  feature-by-feature (t-test/ANOVA, or covariate-adjusted limma/ANCOVA models) and
  the resulting raw p-values must be FDR-corrected to define significant feature sets.
when_to_use_negative:
- The feature table has not yet passed POMAShiny preprocessing — features with more
  than 20% missing values per study group should be removed and missing values imputed
  before testing, not after correction
- The goal is exploratory data analysis (EDA) rather than hypothesis testing — the
  workflow recommends EDA first, and FDR correction is not part of the EDA panel
- Only a single pre-specified comparison is tested, so no multiplicity correction
  across features is required
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: POMA
  role: R/Bioconductor package whose statistical analysis functions run the per-feature
    univariate tests and support correction outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: POMAShiny
  role: Web interface exposing the statistical analysis panel in which the FDR-corrected
    univariate results are generated
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: limma
  role: Linear-model framework used to fit covariate-adjusted per-feature models whose
    p-values are FDR-corrected in parallel with the plain tests
- name: tidyverse
  role: Used for set operations and table assembly when comparing significant feature
    sets and writing the comparison table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:78ec07f9f682c5c36b90d9b5c9ca8825d51507e474f3198728606a2d27406ede
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMA
  tools:
  - POMA
  - POMAShiny
  - R
  - limma
  - tidyverse
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# multiple-testing-correction-fdr

## Summary

Applies multiple-testing (FDR) correction to per-feature univariate p-values from metabolomics/proteomics data analyzed with POMA/POMAShiny, defining significant feature sets at a chosen FDR threshold and enabling comparison of unadjusted versus covariate-adjusted (ANCOVA/limma) significance results.

## When to use

Use when a metabolomics or proteomics feature table (CSV) has been tested feature-by-feature (t-test/ANOVA, or covariate-adjusted limma/ANCOVA models) and the resulting raw p-values must be FDR-corrected to define significant feature sets.

## When NOT to use

- The feature table has not yet passed POMAShiny preprocessing — features with more than 20% missing values per study group should be removed and missing values imputed before testing, not after correction
- The goal is exploratory data analysis (EDA) rather than hypothesis testing — the workflow recommends EDA first, and FDR correction is not part of the EDA panel
- Only a single pre-specified comparison is tested, so no multiplicity correction across features is required

## Inputs

- Preprocessed feature table (CSV) with rows as metabolomics/proteomics features (e.g., ST000284 targeted metabolomics three-group study)
- Target/metadata file (CSV) mapping samples to study groups and covariates
- Chosen FDR significance threshold
- Optional design specification of study covariates for ANCOVA/limma-adjusted models

## Outputs

- Per-feature table of raw and FDR-adjusted p-values for each tested model (unadjusted and covariate-adjusted)
- Significant feature set at the chosen FDR threshold
- Covariate comparison table (covariate_comparison_table.csv) with overlapping, unadjusted-only, and adjusted-only significant features

## How to apply

Load the preprocessed feature table and target (metadata) CSV files — POMAShiny requires exactly these two comma-separated value inputs — and ensure preprocessing (imputation, normalization) and EDA are complete, since EDA is recommended before any statistical analysis. Run per-feature univariate group-comparison tests (t-test or ANOVA) using the POMA R/Bioconductor package's statistical functions, which underlie POMAShiny's statistical analysis panel. Apply multiple-testing correction to the raw p-value vector of every tested feature and define the significant feature set at the chosen FDR threshold. When evaluating covariate adjustment, correct both the plain univariate p-values and the p-values from covariate-combining models (ANCOVA or the limma linear-model framework), then use tidyverse set operations to build an overlap table partitioning features into overlapping, unadjusted-only, and adjusted-only significance. The paper claims that covariate-combining methods such as ANCOVA or limma improve the accuracy and understanding of the results.

## Related tools

- **POMA** (R/Bioconductor package whose statistical analysis functions run the per-feature univariate tests and support correction outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **POMAShiny** (Web interface exposing the statistical analysis panel in which the FDR-corrected univariate results are generated) — https://github.com/nutrimetabolomics/POMAShiny
- **limma** (Linear-model framework used to fit covariate-adjusted per-feature models whose p-values are FDR-corrected in parallel with the plain tests)
- **tidyverse** (Used for set operations and table assembly when comparing significant feature sets and writing the comparison table)

## Examples

```
BiocManager::install("POMA"); shiny::runApp(appDir = "app")
```

## Evaluation signals

- The adjusted-p table has exactly one entry per tested feature, matching the row count of the input feature table
- Adjusted p-values are monotonically greater than or equal to raw p-values for every feature (correction never increases significance)
- The three sets in covariate_comparison_table.csv (overlap, unadjusted-only, adjusted-only) are disjoint and their union equals the union of the two significant feature sets
- Re-running the same tests and correction via POMA in R reproduces the POMAShiny statistical analysis panel results, exploiting the package's reproducibility outside the web environment
- No feature appears significant without a corresponding FDR-adjusted p-value at or below the chosen threshold

## Limitations

- No software version number is stated for POMAShiny or POMA, and no benchmarking or quantitative evaluation of POMAShiny against other tools is reported, so corrected results cannot be cross-validated against a reference implementation
- Local POMAShiny deployment requires the old bs4Dash version 0.5.0 and is not compatible with newer versions due to substantial API breaking changes
- Correction is only meaningful after POMAShiny's two-file CSV input (target/metadata plus features) has been preprocessed; default feature removal uses a 20% missing-value threshold per study group

## Evidence

- [abstract] POMAShiny statistical workflow context: "POMAShiny is a web-based tool that provides a structured, flexible and user-friendly workflow for the visualization, exploration and statistical analysis of metabolomics and proteomics data"
- [results] Input format requirement: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] EDA before statistical analysis: "it is highly recommended to perform an EDA before any statistical analysis"
- [readme] Statistical analysis panel position in the workflow: "POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels."
- [readme] Version pinning limitation for local deployment: "POMAShiny was built with `bs4Dash` version 0.5.0 and is not compatible with newer versions of `bs4Dash` due to substantial breaking changes in the API."
