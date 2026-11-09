---
name: fold-change-computation
description: Use when sex-stratified tumor-vs-normal metabolomics expression data
  are loaded in the massdataset framework and per-feature fold change plus univariate
  significance statistics are needed to build volcano plots.
when_to_use_negative:
- Input is raw LC-MS data (e.g. mzML) that still needs peak picking, data cleaning,
  or quality control — tidyMass raw-data-processing and filter steps must run first
  before fold-change testing
- The input is already a finished differential-expression table with fold change and
  adjusted p-values — no recomputation is needed, only downstream visualization
- The experimental design is not a two-group (tumor vs normal) comparison, e.g. multi-factor
  or longitudinal designs requiring a different statistical model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: massstat
  role: Performs the univariate statistical testing (t-test, Wilcoxon rank-sum) and
    fold-change computation used to build the volcano-plot source tables
  repo: https://www.tidymass.org/
- name: massdataset
  role: Provides the mass_dataset object that stores the processed metabolomics data
    loaded for sex-stratified testing
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidyMass
  role: R-based computational framework whose modular architecture hosts the statistical
    analysis stage and whose mass_dataset format underpins the data structure
  repo: https://github.com/tidymass/tidymass
- name: dplyr
  role: Used in R for compiling and manipulating the per-feature statistics tables
- name: purrr
  role: Used in R for iterating the per-feature univariate testing across the male
    and female cohorts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:502353f1b8a98dae033736c742c424f61d70f096621ab616ba6eaa127fc5f7d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dplyr
  - massdataset
  - massstat
  - purrr
  - tidyMass
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fold-change-computation

## Summary

Computes per-feature fold change between tumor and normal sample groups (separately for male and female CRC cohorts) within the tidyMass/massStat statistical-analysis stage, producing volcano-plot source tables with fold change, p-value, and adjusted p-value columns.

## When to use

Use when sex-stratified tumor-vs-normal metabolomics expression data are loaded in the massdataset framework and per-feature fold change plus univariate significance statistics are needed to build volcano plots.

## When NOT to use

- Input is raw LC-MS data (e.g. mzML) that still needs peak picking, data cleaning, or quality control — tidyMass raw-data-processing and filter steps must run first before fold-change testing
- The input is already a finished differential-expression table with fold change and adjusted p-values — no recomputation is needed, only downstream visualization
- The experimental design is not a two-group (tumor vs normal) comparison, e.g. multi-factor or longitudinal designs requiring a different statistical model

## Inputs

- sex-stratified (male/female) tumor vs normal CRC sample expression/feature intensity table
- mass_dataset object from the massdataset package holding processed metabolomics features and sample metadata
- group labels assigning each sample to tumor or normal within each sex cohort

## Outputs

- male-cohort per-feature statistics table (mean intensity per group, fold change, p-value, adjusted p-value)
- female-cohort per-feature statistics table (mean intensity per group, fold change, p-value, adjusted p-value)
- volcano-plot source tables saved as output files for Fig. 4a and Supplementary Fig. 7

## How to apply

Load the sex-stratified tumor vs normal sample expression data into the tidyMass mass_dataset object structure (the kernel of the object-oriented workflow), then use massStat to run univariate testing on each feature. For each sex cohort separately, compute the fold change between tumor and normal group mean intensities and apply both a t-test and a Wilcoxon rank-sum test per feature. Adjust p-values for multiple testing across all features. Compile per-feature statistics (mean intensity per group, fold change, p-value, adjusted p-value) into one volcano-plot source table per sex, saved as output files for rendering Fig. 4a and Supplementary Fig. 7. Before testing, features should already have passed the tidyMass quality filters (removed if >20% missing values in QC samples or >50% missing values in all study groups).

## Related tools

- **massstat** (Performs the univariate statistical testing (t-test, Wilcoxon rank-sum) and fold-change computation used to build the volcano-plot source tables) — https://www.tidymass.org/
- **massdataset** (Provides the mass_dataset object that stores the processed metabolomics data loaded for sex-stratified testing) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidyMass** (R-based computational framework whose modular architecture hosts the statistical analysis stage and whose mass_dataset format underpins the data structure) — https://github.com/tidymass/tidymass
- **dplyr** (Used in R for compiling and manipulating the per-feature statistics tables)
- **purrr** (Used in R for iterating the per-feature univariate testing across the male and female cohorts)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- Both volcano-plot source tables contain fold-change, p-value, and adjusted p-value columns for all tested features
- P-values are adjusted for multiple comparisons across features in each cohort
- Statistics are computed separately for the male and female cohorts, matching the two tables needed for Fig. 4a and Supplementary Fig. 7
- Fold-change direction is consistent with the group mean intensities (positive when tumor mean exceeds normal mean)
- Features removed as noisy (>20% missing values in QC samples or >50% missing values in study groups) do not appear in the output tables

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, so workflows outside its supported statistical tests may require converting the mass_dataset to other data structures
- Only annotations with level 2 were used for subsequent analysis, and redundant annotations were removed based on the annotation score, so fold-change results are limited to retained level-2 annotations
- The univariate approach (t-test and Wilcoxon) captures univariate tumor-vs-normal differences per feature only; multivariate or covariate-adjusted effects need different methods

## Evidence

- [full_text] massStat univariate testing on the massdataset framework: [evidence span withheld — anchor status: not_found]
- [intro] statistical analysis as a tidyMass workflow stage: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis"
- [intro] object-oriented data structure: "the tidyMass is an object-oriented workflow"
- [intro] upstream feature filtering thresholds: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [readme] tidyMass installation: "remotes::install_gitlab("tidymass/tidymass")"
