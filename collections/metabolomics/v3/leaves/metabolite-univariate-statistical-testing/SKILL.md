---
name: metabolite-univariate-statistical-testing
description: Use when a processed LC-MS feature table (samples x metabolite features
  with group labels) must be tested feature-by-feature for differential abundance
  between two classes, producing fold change, p-value, and adjusted p-value per feature
  for volcano plots.
when_to_use_negative:
- Input is raw LC-MS vendor data still needing peak picking and grouping — that preprocessing
  stage belongs to tools like MZmine, MS-DIAL, or XCMS, not to statistical testing
- The feature table has not been quality-controlled (e.g., noisy features with >20%
  missing values in QC samples or >50% MVs in study groups not yet removed), which
  would bias univariate tests
- The analysis goal is multivariate modeling or biological function mining rather
  than per-feature two-group differential testing
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: massstat
  role: Performs the univariate statistical testing (t-test, Wilcoxon rank-sum, fold
    change) and multiple-testing correction producing volcano-plot statistics within
    the tidyMass ecosystem
  repo: https://www.tidymass.org/
- name: massdataset
  role: Provides the uniform mass_dataset object that carries the processed feature
    intensities and sample metadata through the statistical analysis
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Umbrella R package organizing the tidyMass project packages (including massstat
    and massdataset) used for the statistical analysis step
  repo: https://github.com/tidymass/tidymass
- name: dplyr
  role: Data manipulation for compiling per-feature statistics into the volcano-plot
    source tables
- name: purrr
  role: Iteration over features/cohorts when compiling per-feature test statistics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:34e1a05f87a191d2014dc9dd3444661a90e27a44729b16a31ae76ec4e1872722
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolite-univariate-statistical-testing

## Summary

Per-feature univariate statistical testing of metabolomics intensity data to identify differentially expressed metabolites between two groups (e.g., tumor vs normal), using fold change, t-test/Wilcoxon rank-sum tests, and multiple-testing correction within the tidyMass/massStat framework, with results compiled into volcano-plot source tables.

## When to use

Use when a processed LC-MS feature table (samples x metabolite features with group labels) must be tested feature-by-feature for differential abundance between two classes, producing fold change, p-value, and adjusted p-value per feature for volcano plots.

## When NOT to use

- Input is raw LC-MS vendor data still needing peak picking and grouping — that preprocessing stage belongs to tools like MZmine, MS-DIAL, or XCMS, not to statistical testing
- The feature table has not been quality-controlled (e.g., noisy features with >20% missing values in QC samples or >50% MVs in study groups not yet removed), which would bias univariate tests
- The analysis goal is multivariate modeling or biological function mining rather than per-feature two-group differential testing

## Inputs

- Sex-stratified (or otherwise group-stratified) tumor vs normal sample-by-feature metabolite intensity table (processed LC-MS feature table)
- Sample metadata with group labels (tumor/normal) and cohort assignment (male/female)

## Outputs

- Per-cohort volcano-plot source tables containing per-feature mean intensity per group, fold change, p-value, and adjusted p-value
- List of differentially expressed metabolites per cohort supporting volcano plots such as Fig. 4a and Supplementary Fig. 7

## How to apply

Load the group-labeled expression data into the massdataset/massStat framework so each feature's intensities travel with sample metadata. For each cohort (e.g., the male CRC patients first), compute per-feature fold change between the two groups (tumor vs normal) and apply both a t-test and a Wilcoxon rank-sum test on the intensity distributions. Adjust the resulting p-values for multiple testing across all features, then repeat the identical univariate testing and correction for the second cohort (female patients). Compile per-feature statistics — mean intensity per group, fold change, p-value, adjusted p-value — into one volcano-plot source table per cohort and save them for downstream rendering (e.g., Fig. 4a and Supplementary Fig. 7). Validation requires that both tables carry fold-change, p-value and adjusted p-value columns for every tested feature, with p-values adjusted for multiple comparisons and statistics consistent with the published volcano plots.

## Related tools

- **massstat** (Performs the univariate statistical testing (t-test, Wilcoxon rank-sum, fold change) and multiple-testing correction producing volcano-plot statistics within the tidyMass ecosystem) — https://www.tidymass.org/
- **massdataset** (Provides the uniform mass_dataset object that carries the processed feature intensities and sample metadata through the statistical analysis) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Umbrella R package organizing the tidyMass project packages (including massstat and massdataset) used for the statistical analysis step) — https://github.com/tidymass/tidymass
- **dplyr** (Data manipulation for compiling per-feature statistics into the volcano-plot source tables)
- **purrr** (Iteration over features/cohorts when compiling per-feature test statistics)

## Examples

```
remotes::install_gitlab("tidymass/tidymass"); library(massdataset); library(massstat)
```

## Evaluation signals

- Both volcano-plot source tables contain fold-change, p-value, and adjusted p-value columns for all tested features
- Reported p-values are adjusted for multiple testing across features, not raw
- Compiled statistics (mean intensity per group, fold change, p-value) reproduce the differential metabolites shown in Fig. 4a and Supplementary Fig. 7
- Each cohort (male and female) has its own completed table from an identically parameterized testing pipeline
- All features entering testing passed the prior QC filtering (noisy features with >20% MVs in QC or >50% MVs in study groups already removed)

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though mass_dataset can be converted to other data structures for external tools
- Wilcoxon and t-test results may disagree on skewed intensity distributions; the workflow records both per feature, so a decision rule for reporting is needed
- Only level 2 annotations were used for subsequent analysis in the case study, so differential features may lack confident metabolite identity

## Evidence

- [intro] high-dimensional workflows require per-feature statistics: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis"
- [intro] tidyMass object-oriented kernel: "the tidyMass is an object-oriented workflow"
- [abstract] uniform data structure carries intensities and parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [readme] tidyMass install command grounding the R environment: "remotes::install_gitlab("tidymass/tidymass")"
