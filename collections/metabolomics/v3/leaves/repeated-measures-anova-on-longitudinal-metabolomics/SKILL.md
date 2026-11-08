---
name: repeated-measures-anova-on-longitudinal-metabolomics
description: Use when metabolomics values are measured repeatedly within the same
  subjects (e.g., venous vs arterial conditions or multiple time points) and the goal
  is to rank metabolites by ANOVA significance for feature selection.
when_to_use_negative:
- Measurements are independent across groups rather than repeated within the same
  subjects — MeTEor also offers Mixed ANOVA, Linear Mixed Models, and the Friedman
  test for other designs
- The data is still in wide format with unaddressed missing values — first impute
  and convert to long format before running the ANOVA module
- The goal is binary classification rather than feature ranking — use MeTEor's prediction
  module (logistic regression, random forest, XGBoost) instead
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MeTEor
  role: Provides the Repeated Measures ANOVA module and Configurator used to subset,
    analyze, and rank metabolites for feature selection
  repo: https://github.com/scibiome/meteor
- name: R Shiny
  role: Interactive app framework in which the MeTEor Repeated Measures ANOVA module
    is run
  repo: https://github.com/scibiome/meteor
- name: R
  role: Runtime environment; MeTEor is installed and launched as an R package (also
    available as a Docker image)
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:201147dcfac6ad7626e54d2b54a21b2774d76b43d1863193d4e374f292d2a804
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - R
  - R Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# repeated-measures-anova-on-longitudinal-metabolomics

## Summary

Runs MeTEor's Repeated Measures ANOVA module to rank metabolites by significance for feature selection in longitudinal metabolomics data with within-subject measurements (e.g., venous vs arterial samples across COVID-19 time points), and compares the top hits against prior reported biomarkers.

## When to use

Use when metabolomics values are measured repeatedly within the same subjects (e.g., venous vs arterial conditions or multiple time points) and the goal is to rank metabolites by ANOVA significance for feature selection.

## When NOT to use

- Measurements are independent across groups rather than repeated within the same subjects — MeTEor also offers Mixed ANOVA, Linear Mixed Models, and the Friedman test for other designs
- The data is still in wide format with unaddressed missing values — first impute and convert to long format before running the ANOVA module
- The goal is binary classification rather than feature ranking — use MeTEor's prediction module (logistic regression, random forest, XGBoost) instead

## Inputs

- Longitudinal metabolomics dataset in long format compatible with MeTEor (e.g., covid_data shipped with the MeTEor package)
- Metabolite values of "numeric" data type with a subject identifier, a condition/time factor (venous vs arterial), and a diagnosis/category label
- List of reference top metabolites for overlap comparison (e.g., RI1028.76, RI2473.8, RI3150.76, RI1021.65 from More et al. 2022)

## Outputs

- Ranked metabolite significance table written to ranova_results.csv
- Overlap list of MeTEor-ranked top metabolites vs the four most significant metabolites reported by More et al. 2022 (expected overlap: 4 of 6)

## How to apply

Load a longitudinal metabolomics dataset compatible with MeTEor, e.g. the package-shipped covid_data via data("covid_data"), ensuring metabolite values are of "numeric" type. In the MeTEor Configurator, subset to the relevant category (e.g., COVID-19-infected diagnosis) and relevant time points; if the data is wide format, transform it to long format for MeTEor compatibility and address missing values via imputation first. Run the Repeated Measures ANOVA module on the metabolite values across the within-subject measurement conditions (venous vs arterial), then rank metabolites by their ANOVA significance to perform feature selection. Compare the top-ranked metabolites against the reference hits reported by More et al. 2022 (RI1028.76, RI2473.8, RI3150.76, RI1021.65) and record the overlap. Write the ranked significance table to ranova_results.csv.

## Related tools

- **MeTEor** (Provides the Repeated Measures ANOVA module and Configurator used to subset, analyze, and rank metabolites for feature selection) — https://github.com/scibiome/meteor
- **R Shiny** (Interactive app framework in which the MeTEor Repeated Measures ANOVA module is run) — https://github.com/scibiome/meteor
- **R** (Runtime environment; MeTEor is installed and launched as an R package (also available as a Docker image)) — https://github.com/scibiome/meteor

## Examples

```
library(MeTEor); meteor()
```

## Evaluation signals

- The ranked output table ranova_results.csv exists and contains metabolites ordered by ANOVA significance
- Overlap with the More et al. 2022 top metabolites recovers 4 of 6 hits: RI1028.76, RI2473.8, RI3150.76, and RI1021.65
- Data fed to the ANOVA module is in long format and metabolite value columns are of numeric type with missing values imputed
- Only the intended category subset (e.g., COVID-19-infected diagnosis at the chosen time points) is included in the model, with venous vs arterial as the within-subject factor

## Limitations

- Feature selection by repeated measures ANOVA captures only how features impact the outcome across multiple measurements; MeTEor offers complementary models (Linear Mixed Models, Mixed ANOVA, Friedman test) that may suit other designs
- Data must be transformed from wide to long format and missing values imputed before analysis in MeTEor
- MeTEor requires a Python installation via a Miniconda environment (rminiconda); on Windows the Miniconda path must avoid spaces (e.g., C:\miniconda_py_r), and the Docker option is recommended

## Evidence

- [methods] Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements: "Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements"
- [methods] The data format is being transformed from wide to long format to make it compatible with MeTEor.: "The data format is being transformed from wide to long format to make it compatible with MeTEor."
- [methods] it is important that metabolite values are variables of the "numeric" type: "it is important that metabolite values are variables of the "numeric" type"
- [readme] **Statistical models:** Linear Mixed Models, Repeated Measures ANOVA, Mixed ANOVA, Friedman test: [evidence span withheld — anchor status: external_doc]
