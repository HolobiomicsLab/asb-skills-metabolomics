---
name: volcano-plot-data-preparation
description: Use when you have a feature-by-sample intensity matrix (mass_dataset)
  with a binary group comparison (e.g., tumor vs normal) and need a volcano-plot source
  table with fold change, p-value, and multiple-testing-adjusted p-value per feature.
when_to_use_negative:
- Input is already a processed feature table with fold change, p-value, and adjusted
  p-value columns — the skill would only duplicate existing statistics
- The comparison is not a binary two-group design (e.g., multi-group or time-course
  data) — univariate two-sample t-test/Wilcoxon with fold change is inappropriate
- The data have not yet passed data cleaning and quality control (e.g., features with
  more than 20% missing values in QC samples or more than 50% MVs in all study groups
  should first be removed as noisy features)
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: massstat
  role: Performs the univariate statistical testing (t-test, Wilcoxon rank-sum) and
    multiple-testing correction on mass_dataset objects to generate volcano-plot statistics
  repo: https://www.tidymass.org/
- name: massdataset
  role: Provides the uniform mass_dataset object class used to store and manage the
    processed metabolomics data before statistical analysis
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Umbrella R package organizing the tidyMass project packages (including massstat
    and massdataset) used for this workflow
  repo: https://github.com/tidymass/tidymass
- name: dplyr
  role: Data manipulation of per-feature statistics tables during compilation of the
    volcano-plot source tables
- name: purrr
  role: Iteration over features/cohorts when computing per-feature univariate statistics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:01eb45ba67460c6ec8e1bdaadad486aad3cf4cab43cc01f195e5b64bd2e40d2a
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
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# volcano-plot-data-preparation

## Summary

Prepares volcano-plot source tables by running sex-stratified univariate testing (t-test and Wilcoxon rank-sum) of tumor vs normal metabolite intensities within the tidyMass massStat/massdataset framework, yielding per-feature fold change, p-value, and adjusted p-value columns. This is the statistical-analysis stage of the tidyMass LC-MS untargeted metabolomics workflow that feeds volcano-plot figures.

## When to use

Use when you have a feature-by-sample intensity matrix (mass_dataset) with a binary group comparison (e.g., tumor vs normal) and need a volcano-plot source table with fold change, p-value, and multiple-testing-adjusted p-value per feature.

## When NOT to use

- Input is already a processed feature table with fold change, p-value, and adjusted p-value columns — the skill would only duplicate existing statistics
- The comparison is not a binary two-group design (e.g., multi-group or time-course data) — univariate two-sample t-test/Wilcoxon with fold change is inappropriate
- The data have not yet passed data cleaning and quality control (e.g., features with more than 20% missing values in QC samples or more than 50% MVs in all study groups should first be removed as noisy features)

## Inputs

- Sex-stratified (or otherwise group-stratified) sample expression matrix: features (metabolites) x samples, with tumor vs normal group labels per sample
- mass_dataset object from the massdataset package, or data convertible to it via massdataset's interoperability converters

## Outputs

- Per-group volcano-plot source tables (one for male cohort, one for female cohort) containing, for every tested feature: mean intensity per group, fold change (tumor/normal), raw p-value, and multiple-testing-adjusted p-value
- Saved output files for downstream volcano-plot rendering (e.g., Fig. 4a and Supplementary Fig. 7)

## How to apply

Load the group-stratified (e.g., sex-stratified male and female cohorts) tumor vs normal sample expression data into the massStat/massdataset framework, tidyMass's object-oriented data structure. For each cohort, apply univariate testing on every feature — both a t-test and a Wilcoxon rank-sum test — computing the fold change between tumor and normal groups from per-group mean intensities. Adjust p-values for multiple testing across all tested features, then compile mean intensity per group, fold change, p-value, and adjusted p-value into a per-feature table for each group. Save the two tables as output files for downstream volcano-plot rendering (Fig. 4a, Supplementary Fig. 7 in the CRC case study). The modular tidyMass architecture means each step (cleaning, quality control, statistics) is a separate package stage operating on the same uniform mass_dataset object, keeping the analysis traceable and reproducible.

## Related tools

- **massstat** (Performs the univariate statistical testing (t-test, Wilcoxon rank-sum) and multiple-testing correction on mass_dataset objects to generate volcano-plot statistics) — https://www.tidymass.org/
- **massdataset** (Provides the uniform mass_dataset object class used to store and manage the processed metabolomics data before statistical analysis) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Umbrella R package organizing the tidyMass project packages (including massstat and massdataset) used for this workflow) — https://github.com/tidymass/tidymass
- **dplyr** (Data manipulation of per-feature statistics tables during compilation of the volcano-plot source tables)
- **purrr** (Iteration over features/cohorts when computing per-feature univariate statistics)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- Both volcano-plot source tables contain fold-change, p-value, and adjusted p-value columns for all tested features
- P-values are adjusted for multiple comparisons across features (adjusted p-value column populated, not raw-only)
- Statistics match those reported in Fig. 4a and Supplementary Fig. 7 of the CRC case study
- Each tested feature has per-group mean intensities from which the fold change is derived, so the fold-change column is consistent with the group means
- One table per cohort (male, female) exists, confirming the sex-stratified analysis was run separately for each

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; conversion of the mass_dataset class to other tools' data structures is the workaround
- The article's case-study filter removed features with more than 20% missing values in QC samples or more than 50% MVs in all study groups, and only level-2 annotations were used — statistics on unfiltered data will not match the published figures

## Evidence

- [abstract] uniform data format designed to store and manage processed metabolomics data and processing parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [intro] statistical analysis as a workflow step: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis"
- [intro] interoperability of mass_dataset with other tools: [evidence span withheld — anchor status: not_found]
- [readme] installation of tidymass from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [intro] filtering of noisy features before statistics: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
