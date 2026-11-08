---
name: cross-study-metabolite-overlap-comparison
description: 'Use when the goal is to reproduce/validate a prior metabolomics finding:
  paired within-subject measurements across conditions (e.g., venous vs arterial blood
  over time points) plus a published list of top significant metabolites to compare
  against.'
when_to_use_negative:
- The data are unprocessed raw metabolomics data — MeTEor expects pre-processed data
  in long format; start with the data-cleaning/imputation workflow instead (raw_example_data
  handling is separate).
- The analysis is a between-subjects comparison without repeated within-subject measurements
  — repeated measures ANOVA is inappropriate; consider MeTEor's mixed ANOVA, linear
  mixed models, or Friedman test modules instead.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3272
tools:
- name: MeTEor
  role: R Shiny app providing the Configurator for data subsetting and the Repeated
    Measures ANOVA module used for feature selection on covid_data
  repo: https://github.com/scibiome/meteor
- name: R Shiny
  role: Interactive app framework in which the ANOVA feature selection and ranking
    workflow is executed
  repo: https://github.com/scibiome/meteor
- name: R
  role: Package runtime used to load covid_data, launch the app, and write ranova_results.csv
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:7c46659d6dbf47ce9247c777da22c7e30f499b848a458e8fbcfb68de6f511b06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - R
  - R Shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-study-metabolite-overlap-comparison

## Summary

Runs a repeated measures ANOVA in MeTEor on longitudinal metabolomics data (here the covid_data dataset of venous vs arterial samples from COVID-19 patients), ranks metabolites by significance for feature selection, and compares the top-ranked features against the most significant metabolites reported in a prior study (More et al. 2022) to quantify replication overlap.

## When to use

Use when the goal is to reproduce/validate a prior metabolomics finding: paired within-subject measurements across conditions (e.g., venous vs arterial blood over time points) plus a published list of top significant metabolites to compare against.

## When NOT to use

- The data are unprocessed raw metabolomics data — MeTEor expects pre-processed data in long format; start with the data-cleaning/imputation workflow instead (raw_example_data handling is separate).
- The analysis is a between-subjects comparison without repeated within-subject measurements — repeated measures ANOVA is inappropriate; consider MeTEor's mixed ANOVA, linear mixed models, or Friedman test modules instead.

## Inputs

- covid_data dataset shipped with the MeTEor R package (longitudinal metabolomics, wide format, venous/arterial measurements from COVID-19 and non-COVID-19 pneumonia patients)
- Published list of top significant metabolites from the reference study (More et al. 2022: RI1028.76, RI2473.8, RI3150.76, RI1021.65)

## Outputs

- Ranked metabolite significance table written to ranova_results.csv
- Overlap set between MeTEor ANOVA top-ranked metabolites and the prior study's most significant metabolites (4 of 6 expected here)

## How to apply

Load the package-shipped covid_data dataset via data("covid_data") and open the MeTEor Shiny app (meteor()). In the MeTEor Configurator, subset to the COVID-19-infected diagnosis category for the relevant time points, ensuring data are in long format and metabolite values are numeric (impute missing values first, e.g. with the VIM/laeken-based imputation module). Run the Repeated Measures ANOVA module on metabolite values across the venous versus arterial measurement conditions within the same subjects, since repeated measures ANOVA is used for feature selection by analyzing how different features impact the outcome variable across multiple measurements. Rank metabolites by ANOVA significance, then compare the top-ranked metabolites against the four most significant metabolites from More et al. 2022 (RI1028.76, RI2473.8, RI3150.76, RI1021.65) and record the overlap; export the ranked table to ranova_results.csv.

## Related tools

- **MeTEor** (R Shiny app providing the Configurator for data subsetting and the Repeated Measures ANOVA module used for feature selection on covid_data) — https://github.com/scibiome/meteor
- **R Shiny** (Interactive app framework in which the ANOVA feature selection and ranking workflow is executed) — https://github.com/scibiome/meteor
- **R** (Package runtime used to load covid_data, launch the app, and write ranova_results.csv) — https://github.com/scibiome/meteor

## Examples

```
library(MeTEor); data("covid_data"); meteor()
```

## Evaluation signals

- The ranked table ranova_results.csv contains metabolites ranked by repeated measures ANOVA significance.
- The overlap with More et al. 2022's top metabolites is 4 of 6: RI1028.76, RI2473.8, RI3150.76, and RI1021.65 appear among the MeTEor top-ranked features.
- Input data were verified to be long format with metabolite values of 'numeric' type and missing values imputed before running the ANOVA module.
- Data were subset via the Configurator to the COVID-19-infected diagnosis category for the relevant time points, not the full covid_data table.

## Limitations

- Only partial replication is expected: MeTEor's repeated measures ANOVA recovered 4 of the 6 most significant metabolites from the prior study, so a non-perfect overlap does not indicate a failed run.
- The MetaboAnalyst API was unavailable at time of writing, so the authors developed a workaround for ID conversion; pathway-level ID mapping steps may need this workaround.
- Local R installation requires a Miniconda environment via rminiconda (with a Windows-specific C:\miniconda_py_r path workaround if the install path contains spaces); the Docker container (gordomics/meteor) is the recommended alternative.

## Evidence

- [abstract] replicated prior COVID-19 metabolomics analysis: "we replicated the analysis of metabolomics data from a previously published study on COVID-19 patients"
- [methods] repeated measures ANOVA for feature selection: "Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements"
- [readme] MeTEor statistical methods overview: [evidence span withheld — anchor status: external_doc]
- [other] overlap finding from source card: [evidence span withheld — anchor status: not_found]
