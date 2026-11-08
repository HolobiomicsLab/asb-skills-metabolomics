---
name: within-subject-correlation-statistical-testing
description: Use when the same subjects contribute paired/longitudinal metabolomics
  measurements (e.g., venous and arterial samples at matched time points) and you
  need to rank metabolites by significance for within-subject differential analysis
  or feature selection.
when_to_use_negative:
- Measurements are from independent, unrelated groups with no within-subject pairing
  — use a between-groups model (e.g., Mixed ANOVA) instead
- The metabolite values are not numeric or the data are still in wide format — checking
  data types and wide-to-long transformation must happen first
- The dataset has unaddressed missing values — imputation is required before analysis
  in MeTEor
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_0091
tools:
- name: MeTEor
  role: R Shiny app whose Repeated Measures ANOVA module performs the within-subject
    statistical testing and feature selection
  repo: https://github.com/scibiome/meteor
- name: R
  role: Runtime for the MeTEor package and for exporting the ranked results table
    (write.csv)
  repo: https://github.com/scibiome/meteor
- name: R Shiny
  role: UI framework through which the Repeated Measures ANOVA module and Configurator
    are operated
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:72e886bbf9de0b3e3fe10b7aa28b90fc166b1397818586ea9251cd159cea1a8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - R
  - R Shiny
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# within-subject-correlation-statistical-testing

## Summary

Statistical testing of metabolite features across multiple paired measurements taken within the same subjects — implemented in MeTEor's Repeated Measures ANOVA module — used for feature selection and to identify metabolites differing between within-subject conditions (e.g., venous vs arterial blood in the covid_data study).

## When to use

Use when the same subjects contribute paired/longitudinal metabolomics measurements (e.g., venous and arterial samples at matched time points) and you need to rank metabolites by significance for within-subject differential analysis or feature selection.

## When NOT to use

- Measurements are from independent, unrelated groups with no within-subject pairing — use a between-groups model (e.g., Mixed ANOVA) instead
- The metabolite values are not numeric or the data are still in wide format — checking data types and wide-to-long transformation must happen first
- The dataset has unaddressed missing values — imputation is required before analysis in MeTEor

## Inputs

- Long-format metabolomics dataset with subject, condition/time, and numeric metabolite intensity columns (e.g., MeTEor covid_data)
- MeTEor Configurator subset definition (diagnosis category, relevant time points)

## Outputs

- Ranked metabolite significance table from repeated measures ANOVA (e.g., ranova_results.csv)
- List of top-ranked differential metabolites for overlap comparison against a reference study

## How to apply

Load the paired longitudinal metabolomics data (e.g., MeTEor's covid_data via data("covid_data")) and, if needed, use the MeTEor Configurator to subset to the relevant diagnosis category and time points. Ensure the data are in long format with metabolite values as numeric variables, so each subject's venous and arterial measurements can be paired within the Repeated Measures ANOVA module. Run the module so that feature impact on the outcome variable is assessed across the multiple within-subject measurements rather than between independent groups. Rank metabolites by ANOVA significance (feature selection) and export the ranked table (e.g., ranova_results.csv); here this recovered four of the six most significant metabolites from the More et al. 2022 COVID-19 pneumonia comparison: RI1028.76, RI2473.8, RI3150.76, and RI1021.65.

## Related tools

- **MeTEor** (R Shiny app whose Repeated Measures ANOVA module performs the within-subject statistical testing and feature selection) — https://github.com/scibiome/meteor
- **R** (Runtime for the MeTEor package and for exporting the ranked results table (write.csv)) — https://github.com/scibiome/meteor
- **R Shiny** (UI framework through which the Repeated Measures ANOVA module and Configurator are operated) — https://github.com/scibiome/meteor

## Examples

```
library(MeTEor); data("covid_data"); meteor()
```

## Evaluation signals

- Top-ranked metabolites overlap with the reference list from More et al. 2022 — expect recovery of RI1028.76, RI2473.8, RI3150.76, and RI1021.65 (4 of 6)
- All metabolite value columns are of "numeric" type before running the module
- Each subject's venous and arterial measurements are analyzed as paired within-subject observations, not as independent samples
- Ranked significance table (ranova_results.csv) is produced with metabolite IDs and ANOVA significance values

## Limitations

- MeTEor's replication recovered only 4 of the 6 most significant metabolites from the original COVID-19 study, so partial overlap with published results is an expected outcome
- Repeated measures ANOVA is one of several statistical models in MeTEor (Linear Mixed Models, Mixed ANOVA, Friedman test); model choice depends on the design
- Missing values must be imputed beforehand, and MetaboAnalyst API unavailability required a workaround for ID-related steps

## Evidence

- [methods] Repeated measures ANOVA for within-subject feature selection: "Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements"
- [readme] Statistical models available in MeTEor: [evidence span withheld — anchor status: external_doc]
- [other] Finding from the source task card: 4 of 6 overlap: [evidence span withheld — anchor status: not_found]
