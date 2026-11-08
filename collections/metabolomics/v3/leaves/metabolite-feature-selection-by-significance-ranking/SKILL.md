---
name: metabolite-feature-selection-by-significance-ranking
description: Use when a longitudinal metabolomics dataset (long-format, metabolite
  values as numeric variables measured repeatedly within subjects) must be reduced
  to the most outcome-relevant metabolites before prediction or comparison with previously
  reported biomarkers.
when_to_use_negative:
- The metabolite values are not numeric or the data are still in wide format — MeTEor
  requires numeric-type metabolite variables and long-format data before the ANOVA
  module can run
- Each metabolite is measured only once per subject — repeated measures ANOVA requires
  multiple measurements across conditions/time points within subjects; use a different
  statistical model (e.g., linear mixed models or Friedman test) instead
- The goal is unsupervised exploration rather than outcome-related feature selection
  — PCA or TCAM dimensionality reduction is the appropriate MeTEor module
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MeTEor
  role: R Shiny app whose Repeated Measures ANOVA module performs the significance-based
    feature selection and whose Configurator subsets data to the relevant diagnosis
    category and time points
  repo: https://github.com/scibiome/meteor
- name: R Shiny
  role: Framework hosting the MeTEor GUI through which the ANOVA module and result
    export are run
  repo: https://github.com/scibiome/meteor
- name: R
  role: Language in which MeTEor is packaged (library(MeTEor)) and in which results
    are written with write.csv
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:74c3e10ebfb3cc10e763716f6cb714e33733e08e3ac68307652a52eb0fe16b34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - R
  - R Shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolite-feature-selection-by-significance-ranking

## Summary

Selects metabolites as features for downstream modeling by ranking them on repeated measures ANOVA significance across within-subject conditions (e.g., venous vs arterial blood in the MeTEor covid_data study), then validates against a reference feature list.

## When to use

Use when a longitudinal metabolomics dataset (long-format, metabolite values as numeric variables measured repeatedly within subjects) must be reduced to the most outcome-relevant metabolites before prediction or comparison with previously reported biomarkers.

## When NOT to use

- The metabolite values are not numeric or the data are still in wide format — MeTEor requires numeric-type metabolite variables and long-format data before the ANOVA module can run
- Each metabolite is measured only once per subject — repeated measures ANOVA requires multiple measurements across conditions/time points within subjects; use a different statistical model (e.g., linear mixed models or Friedman test) instead
- The goal is unsupervised exploration rather than outcome-related feature selection — PCA or TCAM dimensionality reduction is the appropriate MeTEor module

## Inputs

- Long-format metabolomics data frame compatible with MeTEor (e.g., covid_data loaded via data("covid_data")), with numeric metabolite intensity columns (RI-prefixed features like RI1028.76) and subject/diagnosis/condition metadata
- Subsetted data selection made in the MeTEor Configurator (diagnosis category and time points of interest)
- Optional reference list of significant metabolites from prior work (e.g., RI1028.76, RI2473.8, RI3150.76, RI1021.65 from More et al. 2022) for overlap checking

## Outputs

- Ranked metabolite significance table from the repeated measures ANOVA (e.g., exported to ranova_results.csv)
- List of top-ranked (selected) metabolite features
- Overlap record between MeTEor's top-ranked metabolites and the reference metabolites from prior published work

## How to apply

Load the data into MeTEor (e.g., data('covid_data') shipped with the R package) and ensure it is in long format with numeric metabolite values, using the Configurator to subset to the condition of interest (e.g., the COVID-19-infected diagnosis category at the relevant time points). Open the Repeated Measures ANOVA module and run it on the metabolite values across the within-subject measurement conditions (venous vs arterial samples of the same subjects), since repeated measures ANOVA is used for feature selection by analyzing how different features impact the outcome variable across multiple measurements. Rank metabolites by their ANOVA significance and take the top-ranked ones as selected features. Validate by comparing the top hits to a reference list — here, four of the six most significant metabolites from More et al. 2022 (RI1028.76, RI2473.8, RI3150.76, RI1021.65) were recovered — and export the ranked table, e.g., to ranova_results.csv via write.csv.

## Related tools

- **MeTEor** (R Shiny app whose Repeated Measures ANOVA module performs the significance-based feature selection and whose Configurator subsets data to the relevant diagnosis category and time points) — https://github.com/scibiome/meteor
- **R Shiny** (Framework hosting the MeTEor GUI through which the ANOVA module and result export are run) — https://github.com/scibiome/meteor
- **R** (Language in which MeTEor is packaged (library(MeTEor)) and in which results are written with write.csv) — https://github.com/scibiome/meteor

## Examples

```
library(MeTEor); data("covid_data"); meteor()  # then run the Repeated Measures ANOVA module on venous vs arterial conditions and export ranked results to ranova_results.csv
```

## Evaluation signals

- The ANOVA runs without errors only after data are long-format and metabolite columns are of "numeric" type — type-check the input frame first
- Ranked table reproduces the known overlap: 4 of 6 top metabolites from More et al. 2022 (RI1028.76, RI2473.8, RI3150.76, RI1021.65) appear among MeTEor's most significant hits
- ranova_results.csv exists and contains one significance-ranked row per metabolite (writable via write.csv(..., file = "ranova_results.csv"))
- The Configurator subset is correct: only the COVID-19-infected diagnosis category at the chosen time points feeds the ANOVA, so selected metabolites should be stable across reruns of the same subset
- Selected metabolites should be biologically interpretable via MeTEor's KEGG pathway mapping / enrichment follow-up

## Limitations

- MeTEor is an R Shiny app requiring a Miniconda environment (rminiconda) for the R-package install; on Windows the Miniconda path must not contain spaces (use C:\miniconda_py_r), and the Docker route is recommended — on macOS the container must run with --platform linux/amd64
- The MetaboAnalyst API was unavailable at publication time, so metabolite ID conversion relies on a manual workaround, which affects the ID mapping step that follows feature selection
- The ANOVA captures features whose differences relate to the outcome across repeated measurements; features significant only in unpaired or single-measurement designs will not be recovered by this module
- No changelog found for the repository, so version-specific behavior differences may not be traceable

## Evidence

- [methods] Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements: "Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements"
- [methods] it is important that metabolite values are variables of the "numeric" type: "it is important that metabolite values are variables of the "numeric" type"
- [methods] The data format is being transformed from wide to long format to make it compatible with MeTEor.: "The data format is being transformed from wide to long format to make it compatible with MeTEor."
- [methods] write.csv(data_meta_imputed_long, file = "data_long.csv", row.names = FALSE ): "write.csv(data_meta_imputed_long, file = "data_long.csv", row.names = FALSE )"
- [abstract] we replicated the analysis of metabolomics data from a previously published study on COVID-19 patients: "we replicated the analysis of metabolomics data from a previously published study on COVID-19 patients"
- [readme] Statistical models: Linear Mixed Models, Repeated Measures ANOVA, Mixed ANOVA, Friedman test: [evidence span withheld — anchor status: external_doc]
