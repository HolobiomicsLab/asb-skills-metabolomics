---
name: multivariate-anova-across-metabolite-columns
description: Use when the input is a preprocessed metabolomics concentration table
  (samples as rows, metabolite columns, a group/factor column) and per-metabolite
  significance testing across two or more sample groups is the goal.
when_to_use_negative:
- The data have only two groups and a univariate non-parametric comparison is wanted
  — Viime pairs the Wilcoxon rank sum test with two-group univariate analysis, and
  ANOVA is designated for two or more groups
- The table is raw instrument output (e.g. unnormalized NMR/MS intensities) that has
  not passed Viime's normalization, transformation, and scaling steps
- The concentration columns contain non-numeric data or missing values exceeding the
  default threshold, which Viime flags as errors that must be corrected (or imputed
  via its MCAR/MNAR strategy) before analysis
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: host application whose univariate/multivariate testing workflow performs multivariate
    ANOVA with post-hoc Tukey on grouped metabolite data
  repo: https://github.com/girder/viime
- name: R
  role: statistics backend running the ANOVA and Tukey HSD functions in the custom
    viime R package under devops/
- name: OpenCPU
  role: internal REST service integrating Python (pandas) with the R statistics functions
  repo: https://www.opencpu.org/
- name: Pandas
  role: parses the concentration CSV and exchanges data frames with R via viime/opencpu.py
- name: Flask
  role: RESTful API backend through which the ANOVA/Tukey processing is exposed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:9212e345746f7a4d47e5376754d44751748aaad7a235b73b77fe9bf91ec704aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - OpenCPU
  - Pandas
  - R
  - Viime
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multivariate-anova-across-metabolite-columns

## Summary

Runs a multivariate ANOVA over metabolite concentration columns in a group-factored dataset and, when two or more groups are present, automatically applies a post-hoc Tukey HSD test yielding per-metabolite p-values for every inter-group comparison, as implemented in Viime via its R/OpenCPU processing backend.

## When to use

Use when the input is a preprocessed metabolomics concentration table (samples as rows, metabolite columns, a group/factor column) and per-metabolite significance testing across two or more sample groups is the goal.

## When NOT to use

- The data have only two groups and a univariate non-parametric comparison is wanted — Viime pairs the Wilcoxon rank sum test with two-group univariate analysis, and ANOVA is designated for two or more groups
- The table is raw instrument output (e.g. unnormalized NMR/MS intensities) that has not passed Viime's normalization, transformation, and scaling steps
- The concentration columns contain non-numeric data or missing values exceeding the default threshold, which Viime flags as errors that must be corrected (or imputed via its MCAR/MNAR strategy) before analysis

## Inputs

- preprocessed metabolomics concentration table as .csv or .xlsx (samples as rows, metabolite concentration columns, a group/factor column)
- group/factor column assignments designating the two or more sample groups to compare

## Outputs

- results table keyed by metabolite with the ANOVA p-value and one Tukey p-value column per pairwise inter-group comparison (anova_tukey_results.csv)
- significantly different metabolites per inter-group comparison, selectable via check box at the top of each column for further analysis

## How to apply

Load the preprocessed concentration table (CSV or .xlsx) with pandas, using the Viime upload schema in which columns can be designated as the primary ID, a factor, the group, or a metabolite concentration column. Split each metabolite concentration column by group and run a one-way ANOVA across the groups using the R statistics stack exposed through Viime's custom R package, which 'should accept a path to a CSV file as an argument and return a data frame'; the pandas-to-R bridge lives in viime/opencpu.py. Because Viime's design applies ANOVA only to data with two or more groups, apply the post-hoc Tukey HSD test automatically to compute pairwise p-values for each inter-group comparison of every metabolite. Assemble a results table keyed by metabolite with the ANOVA p-value plus one Tukey p-value column per pairwise group comparison, and write it to anova_tukey_results.csv. To extend the exposed tests, add the function to the devops/viime R package and rebuild the OpenCPU docker image.

## Related tools

- **Viime** (host application whose univariate/multivariate testing workflow performs multivariate ANOVA with post-hoc Tukey on grouped metabolite data) — https://github.com/girder/viime
- **R** (statistics backend running the ANOVA and Tukey HSD functions in the custom viime R package under devops/)
- **OpenCPU** (internal REST service integrating Python (pandas) with the R statistics functions) — https://www.opencpu.org/
- **Pandas** (parses the concentration CSV and exchanges data frames with R via viime/opencpu.py)
- **Flask** (RESTful API backend through which the ANOVA/Tukey processing is exposed)

## Examples

```
import pandas as pd; df = pd.read_csv('concentrations.csv'); df.to_csv('for_r.csv', index=False)  # push through viime/opencpu.py to the viime R package ANOVA+Tukey function, then save anova_tukey_results.csv
```

## Evaluation signals

- anova_tukey_results.csv exists and is parseable as a table
- every tested metabolite has an ANOVA p-value entry plus one Tukey p-value entry per pairwise group comparison
- all reported p-values fall within [0, 1]
- the number of Tukey columns matches the number of pairwise inter-group comparisons for the dataset's groups
- the R functions called over OpenCPU accept a CSV path and return a data frame, per the viime R package contract

## Limitations

- Multivariate ANOVA with Tukey in Viime applies only to datasets with two or more groups; two-group univariate comparisons use the Wilcoxon rank sum test instead
- The pandas–R bridge in viime/opencpu.py is described in the README as work-in-progress
- Exposing additional statistical methods requires editing the custom R package and rebuilding the OpenCPU docker image; the OpenCPU service must be running (e.g. via docker run -p 8004:8004 viime) for R-backed tests to execute

## Evidence

- [abstract] Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively.: "Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively."
- [abstract] Viime ANOVA/Tukey with automated post-hoc pairwise p-values: "Metabolites that are significantly different in each of the intergroup comparisons can be selected for further analysis with a check box at the top of each column."
- [readme] README: viime R package function contract: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [readme] README: pandas–R bridge location: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames."
- [readme] README: extending exposed R methods: "To add additional methods exposed to the API server, add the function to the viime package and rebuild the docker image."
