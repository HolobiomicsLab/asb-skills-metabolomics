---
name: sex-stratified-group-comparison
description: Use when a LC-MS metabolomics feature table must be compared between
  two conditions (e.g., tumor vs normal) within patient subgroups such as male and
  female cohorts, to generate per-sex differential-metabolite statistics.
when_to_use_negative:
- Input is already a processed differential-statistics table — the fold-change, t-test,
  Wilcoxon, and FDR-adjustment steps are then redundant.
- The analysis goal is a single pooled two-group comparison across all patients without
  sex stratification.
- Data have not been cleaned of noisy features (e.g., >20% missing values in QC samples
  or >50% MVs in study groups), which should be filtered before univariate testing.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: massstat
  role: Performs per-cohort univariate differential testing (fold change, t-test,
    Wilcoxon rank-sum) and multiple-testing correction for volcano-plot statistics
- name: massdataset
  role: Provides the 'mass_dataset' uniform data structure used to hold and manage
    the sex-stratified processed metabolomics data before statistical analysis
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Umbrella R package of the tidyMass project that organizes the packages (massstat,
    massdataset) used for the statistical analysis step
  repo: https://gitlab.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:d6edd84e2815d2527d4079e1223c88d8adefe5987199396408a8dc94e22faff7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dplyr
  - massdataset
  - massstat
  - purrr
  - tidymass
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sex-stratified-group-comparison

## Summary

Detects differentially expressed metabolites between two groups (e.g., tumor vs normal) separately within each sex cohort using massStat/massdataset univariate testing (t-test and Wilcoxon rank-sum), producing volcano-plot source tables per sex.

## When to use

Use when a LC-MS metabolomics feature table must be compared between two conditions (e.g., tumor vs normal) within patient subgroups such as male and female cohorts, to generate per-sex differential-metabolite statistics.

## When NOT to use

- Input is already a processed differential-statistics table — the fold-change, t-test, Wilcoxon, and FDR-adjustment steps are then redundant.
- The analysis goal is a single pooled two-group comparison across all patients without sex stratification.
- Data have not been cleaned of noisy features (e.g., >20% missing values in QC samples or >50% MVs in study groups), which should be filtered before univariate testing.

## Inputs

- Sex-stratified LC-MS feature/sample expression (intensity) table with tumor vs normal group labels
- Sample metadata assigning samples to sex cohort and condition (tumor/normal)

## Outputs

- Per-sex volcano-plot source table with columns for mean intensity per group, fold change, p-value, and adjusted p-value for every tested feature
- Two tables (male cohort and female cohort) suitable for rendering Fig. 4a and Supplementary Fig. 7 volcano plots

## How to apply

Load the sex-stratified tumor vs normal sample expression data into the massdataset 'mass_dataset' object framework, which tidyMass uses as the kernel data structure. For each sex cohort independently, run massStat univariate testing on every feature: compute fold change between tumor and normal groups and apply both a t-test and a Wilcoxon rank-sum test. Adjust p-values for multiple testing across features within each cohort, then compile per-feature statistics (mean intensity per group, fold change, p-value, adjusted p-value) into one volcano-plot source table per sex. Save the two tables for downstream volcano-plot rendering (as in Fig. 4a and Supplementary Fig. 7 of the tidyMass CRC case study). Optionally convert the mass_dataset to other structures first — tidyMass has high interoperability with other existing tools.

## Related tools

- **massstat** (Performs per-cohort univariate differential testing (fold change, t-test, Wilcoxon rank-sum) and multiple-testing correction for volcano-plot statistics)
- **massdataset** (Provides the 'mass_dataset' uniform data structure used to hold and manage the sex-stratified processed metabolomics data before statistical analysis) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Umbrella R package of the tidyMass project that organizes the packages (massstat, massdataset) used for the statistical analysis step) — https://gitlab.com/tidymass/tidymass

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- Both per-sex volcano-plot source tables contain fold-change, p-value, and adjusted p-value columns for all tested features.
- P-values are adjusted for multiple comparisons across features within each cohort before the tables are saved.
- Resulting statistics match the differentially expressed metabolites reported in the Fig. 4a volcano plots and Supplementary Fig. 7.
- Each cohort (male, female) has its own independent set of univariate tests — statistics are not computed on pooled samples.
- Output tables include per-group mean intensities in addition to fold-change and p-value columns.

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, requiring conversion of the 'mass_dataset' class to other data structures.
- The uniform data structure and modular design assume the data passed in are already cleaned and quality-controlled; noisy features (high missingness) must be removed beforehand.

## Evidence

- [intro] massdataset interoperability: [evidence span withheld — anchor status: not_found]
- [intro] statistical analysis step: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis"
- [intro] noisy feature filter: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [readme] tidymass install: "remotes::install_gitlab("tidymass/tidymass")"
