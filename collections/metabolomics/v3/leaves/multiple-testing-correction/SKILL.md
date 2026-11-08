---
name: multiple-testing-correction
description: Use when a feature-by-feature univariate test (e.g. t-test or Wilcoxon
  rank-sum between tumor and normal groups) has been run on many metabolomics features
  and p-values must be adjusted before reporting differential metabolites or rendering
  volcano plots.
when_to_use_negative:
- Input is raw LC-MS data rather than a processed feature table — peak picking and
  data cleaning (e.g. via xcms or tidyMass processing) must happen first
- Features have not been filtered for missing values (noisy features with >20% MV
  in QC or >50% MV in study groups should be removed before testing)
- Only a single pre-specified hypothesis is being tested, so multiplicity correction
  is not applicable
edam_operation: http://edamontology.org/operation_2262
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2269
tools:
- name: massstat
  role: Performs the univariate statistical testing and multiple-testing correction
    that yields the volcano-plot statistics for the tumor-vs-normal comparison
- name: massdataset
  role: Provides the 'mass_dataset' uniform data structure that holds the sex-stratified
    expression data feeding the statistical tests
- name: tidyMass
  role: R-based computational framework whose statistical-analysis stage this correction
    belongs to; ecosystem package installed via remotes
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Language in which the massStat/massdataset workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:8af1a3d6e3ef7cba81c55b3bd4aa3fd7d7a65f1e95c01594485685bd19808073
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - dplyr
  - massdataset
  - massstat
  - purrr
  - tidyMass
  edam_operation: http://edamontology.org/operation_2262
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multiple-testing-correction

## Summary

Adjust per-feature p-values from univariate metabolomics comparisons for the multiplicity incurred when testing thousands of LC-MS features, as done in the tidyMass/massStat CRC case study to generate volcano-plot statistics for sex-stratified tumor-vs-normal differential metabolites.

## When to use

Use when a feature-by-feature univariate test (e.g. t-test or Wilcoxon rank-sum between tumor and normal groups) has been run on many metabolomics features and p-values must be adjusted before reporting differential metabolites or rendering volcano plots.

## When NOT to use

- Input is raw LC-MS data rather than a processed feature table — peak picking and data cleaning (e.g. via xcms or tidyMass processing) must happen first
- Features have not been filtered for missing values (noisy features with >20% MV in QC or >50% MV in study groups should be removed before testing)
- Only a single pre-specified hypothesis is being tested, so multiplicity correction is not applicable

## Inputs

- Sex-stratified tumor vs normal sample feature-intensity matrix (processed peak table from massdataset/mass_dataset object)
- Group/phenotype labels identifying tumor and normal samples within each sex cohort

## Outputs

- Per-sex volcano-plot source table with mean intensity per group, fold change, raw p-value, and adjusted p-value per feature
- Adjusted p-value vector across all tested features for each cohort

## How to apply

Load the sex-stratified tumor-vs-normal expression matrix into the tidyMass 'mass_dataset' object (massdataset package) so processed data and parameters remain traceable. Within each cohort (male, female) run per-feature univariate testing — t-test and Wilcoxon rank-sum test — computing fold change between tumor and normal groups. Correct the resulting p-values for multiple testing across all tested features within each cohort separately, since each sex-stratified analysis constitutes its own hypothesis family. Compile per-feature statistics (mean intensity per group, fold change, raw p-value, adjusted p-value) into the volcano-plot source tables, which in the case study support Fig. 4a and Supplementary Fig. 7. Validation requires that both tables contain fold-change, p-value and adjusted p-value columns for every tested feature and that the statistics reproduce the reported volcano plots.

## Related tools

- **massstat** (Performs the univariate statistical testing and multiple-testing correction that yields the volcano-plot statistics for the tumor-vs-normal comparison)
- **massdataset** (Provides the 'mass_dataset' uniform data structure that holds the sex-stratified expression data feeding the statistical tests)
- **tidyMass** (R-based computational framework whose statistical-analysis stage this correction belongs to; ecosystem package installed via remotes) — https://github.com/tidymass/tidymass
- **R** (Language in which the massStat/massdataset workflow is executed)

## Examples

```
remotes::install_gitlab("tidymass/massstat")
```

## Evaluation signals

- Both volcano-plot source tables contain fold-change, p-value and adjusted p-value columns for all tested features
- P-values are adjusted for multiple comparisons (adjusted p-value ≤ raw p-value for every feature)
- One adjusted table per cohort exists (male and female), matching the sex-stratified design
- Adjusted statistics reproduce the features highlighted in Fig. 4a and Supplementary Fig. 7 volcano plots

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though 'mass_dataset' can be converted to other data structures
- Correction applies per cohort; pooling sexes before adjustment would not match the sex-stratified analysis reported in the case study
- The adjustment is only as valid as upstream filtering — noisy features (e.g. >20% missing values in QC samples) must be removed beforehand

## Evidence

- [other] Differentially expressed metabolites between tumor and normal tissues were detected separately for male and female CRC patients using the massStat package: [evidence span withheld — anchor status: not_found]
- [intro] the tidyMass is an object-oriented workflow: [evidence span withheld — anchor status: not_found]
- [intro] requiring the optimization of multiple steps such as raw data processing: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis"
- [readme] The *xcms* R package provides functionality to efficiently preprocess LC-MS: [evidence span withheld — anchor status: external_doc]
