---
name: shiny-app-configurator-data-subsetting
description: Use when a MeTEor workflow requires analyzing only a subset of a loaded
  metabolomics dataset — e.g., filtering to the COVID-19-infected diagnosis category
  and specific measurement time points of covid_data before running Repeated Measures
  ANOVA or other modules.
when_to_use_negative:
- The data are already restricted to the target group and time points, so Configurator
  subsetting would be redundant
- Data are still in wide format or contain non-numeric metabolite values or unimputed
  missing values — these must be fixed (long-format conversion, type checking, imputation)
  before subsetting is meaningful in MeTEor
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MeTEor
  role: R Shiny app whose Configurator module is used to subset the loaded metabolomics
    data by diagnosis category and time points before statistical analysis
  repo: https://github.com/scibiome/meteor
- name: R Shiny
  role: Framework providing the interactive UI through which the Configurator subsetting
    selections are made
  repo: https://github.com/scibiome/meteor
- name: R
  role: Package environment in which MeTEor runs and from which covid_data is loaded
    via data('covid_data')
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:cac388ad3813b55f47dfc2f70de098ba2683e8e6b69d0dfa82cad9cd691a7117
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - R
  - R Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Shiny App Configurator Data Subsetting

## Summary

Subsets longitudinal metabolomics data within the MeTEor R Shiny app's Configurator (e.g., restricting the covid_data dataset to a specific diagnosis category and relevant time points) so that downstream modules like Repeated Measures ANOVA operate on the intended subject groups.

## When to use

Use when a MeTEor workflow requires analyzing only a subset of a loaded metabolomics dataset — e.g., filtering to the COVID-19-infected diagnosis category and specific measurement time points of covid_data before running Repeated Measures ANOVA or other modules.

## When NOT to use

- The data are already restricted to the target group and time points, so Configurator subsetting would be redundant
- Data are still in wide format or contain non-numeric metabolite values or unimputed missing values — these must be fixed (long-format conversion, type checking, imputation) before subsetting is meaningful in MeTEor

## Inputs

- Long-format metabolomics dataset loaded into MeTEor (e.g., data('covid_data') from the MeTEor package) with numeric metabolite values, subject identifiers, diagnosis categories, and time points
- MeTEor Configurator selections (diagnosis category, e.g., COVID-19-infected; relevant time points/conditions such as venous vs arterial measurements)

## Outputs

- Subsetted long-format metabolomics data table restricted to the chosen diagnosis category and time points
- Downstream ranked metabolite significance table (e.g., ranova_results.csv from the Repeated Measures ANOVA module)

## How to apply

First load the data into MeTEor via the import module (press 'Click for data importing'), ensuring the data have been transformed from wide to long format with numeric metabolite values and missing values imputed. Then use the MeTEor Configurator UI to subset the data — for the covid_data workflow, select the COVID-19-infected diagnosis category and the relevant venous/arterial measurement time points within the same subjects. This subsetting is a prerequisite for the Repeated Measures ANOVA module, which uses the restricted data for feature selection across multiple measurements. Finally, rank the resulting metabolites by significance and export tables such as ranova_results.csv.

## Related tools

- **MeTEor** (R Shiny app whose Configurator module is used to subset the loaded metabolomics data by diagnosis category and time points before statistical analysis) — https://github.com/scibiome/meteor
- **R Shiny** (Framework providing the interactive UI through which the Configurator subsetting selections are made) — https://github.com/scibiome/meteor
- **R** (Package environment in which MeTEor runs and from which covid_data is loaded via data('covid_data'))

## Examples

```
library(MeTEor); data("covid_data"); meteor()
```

## Evaluation signals

- The subsetted table contains only records for the selected diagnosis category (e.g., COVID-19-infected) and the chosen time points, with venous and arterial measurements paired within the same subjects
- Metabolite values remain numeric and no missing values remain after the preceding imputation step
- Repeated Measures ANOVA run on the subset recovers the expected top-ranked metabolites (e.g., four of six overlap with More et al. 2022: RI1028.76, RI2473.8, RI3150.76, RI1021.65)
- The ranked significance table exports successfully as ranova_results.csv without format errors
- Row counts of the subset equal the expected subject × time point combinations for the selected category

## Limitations

- Data must already be in long format, with duplicates removed, irrelevant columns dropped, metabolite values typed as numeric, and missing values imputed before MeTEor analysis — the Configurator does not perform these preprocessing steps
- Subsetting is done through the Shiny UI, so selections are manual and not easily scripted or version-controlled
- MetaboAnalyst's API is currently unavailable, so ID conversion requires the documented workaround, which may affect metabolite labeling in subsets

## Evidence

- [methods] Import module quote: "With the import module you can easily load your data into MeTEor. Just press the button 'Click for data importing'."
- [methods] Long format prerequisite: "The data format is being transformed from wide to long format to make it compatible with MeTEor."
- [methods] ANOVA module: "Repeated measures ANOVA can be used for feature selection by analyzing how different features impact the outcome variable across multiple measurements"
- [readme] MeTEor purpose: [evidence span withheld — anchor status: not_found]
