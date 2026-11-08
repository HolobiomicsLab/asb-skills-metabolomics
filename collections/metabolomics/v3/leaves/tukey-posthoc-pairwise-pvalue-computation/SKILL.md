---
name: tukey-posthoc-pairwise-pvalue-computation
description: Use when a metabolomics concentration table (CSV with sample rows, metabolite
  columns, and a group/factor column) has two or more sample groups and you need,
  per metabolite, an ANOVA p-value plus pairwise Tukey p-values for every inter-group
  comparison.
when_to_use_negative:
- The dataset has only two groups and a nonparametric comparison is intended — Viime
  applies the univariate Wilcoxon rank sum test for two-group data and reserves multivariate
  ANOVA for two or more groups
- Data have not yet been through Viime's preprocessing chain (error correction, MCAR/MNAR
  imputation, normalization, transformation, scaling), so concentration values are
  not comparable across groups
- The goal is model-based multivariate statistics (e.g. PLS-DA) rather than per-metabolite
  group significance testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Viime
  role: Web application whose backend implements the multivariate ANOVA with automatic
    post-hoc Tukey testing on grouped metabolomics data
  repo: https://github.com/girder/viime
- name: R
  role: Provides the specialized statistical algorithms (ANOVA and Tukey HSD) via
    the custom viime R package whose functions accept a CSV path and return a data
    frame
  repo: https://github.com/girder/viime
- name: OpenCPU
  role: Internal REST service providing the R-python integration so the Flask backend
    can invoke R statistical functions
- name: Pandas
  role: Parses the raw CSV and exchanges data frames with R through viime/opencpu.py
- name: Flask
  role: RESTful API backend that drives the statistical testing workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:b2e345ff46734495de99c23a38e3d21656c40667d5b37981bff9e4f7495a9076
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tukey-posthoc-pairwise-pvalue-computation

## Summary

Computes pairwise inter-group p-values with the Tukey HSD post-hoc test following a multivariate ANOVA, reproducing Viime's statistical significance testing for metabolomics concentration data with two or more groups.

## When to use

Use when a metabolomics concentration table (CSV with sample rows, metabolite columns, and a group/factor column) has two or more sample groups and you need, per metabolite, an ANOVA p-value plus pairwise Tukey p-values for every inter-group comparison.

## When NOT to use

- The dataset has only two groups and a nonparametric comparison is intended — Viime applies the univariate Wilcoxon rank sum test for two-group data and reserves multivariate ANOVA for two or more groups
- Data have not yet been through Viime's preprocessing chain (error correction, MCAR/MNAR imputation, normalization, transformation, scaling), so concentration values are not comparable across groups
- The goal is model-based multivariate statistics (e.g. PLS-DA) rather than per-metabolite group significance testing

## Inputs

- Preprocessed metabolomics concentration table as CSV (.csv, also .xlsx accepted by Viime upload) with samples as rows, metabolite concentration columns, a designated group column, and a primary ID column
- Group/factor column assignment designating each sample's group membership (two or more groups)

## Outputs

- anova_tukey_results.csv: a table keyed by metabolite with the ANOVA p-value and one Tukey HSD p-value column per pairwise inter-group comparison
- Set of significantly different metabolites per inter-group comparison, selectable via check boxes for further analysis

## How to apply

Load the preprocessed concentration table with pandas, splitting concentration values by the designated group column so each metabolite can be tested across groups. For each metabolite column, run a one-way ANOVA across the two or more groups via the R statistics stack exposed through Viime's viime package functions, called over the OpenCPU REST bridge implemented in viime/opencpu.py (pandas data frames are exchanged with R data frames there). Automatically apply the post-hoc Tukey HSD test for metabolites with two or more groups to compute a p-value for each pairwise group comparison. Assemble a results table keyed by metabolite containing the ANOVA p-value and one Tukey p-value column per pairwise comparison, and write it to anova_tukey_results.csv. Viime also allows significantly different metabolites in each inter-group comparison to be selected via check boxes for further analysis.

## Related tools

- **Viime** (Web application whose backend implements the multivariate ANOVA with automatic post-hoc Tukey testing on grouped metabolomics data) — https://github.com/girder/viime
- **R** (Provides the specialized statistical algorithms (ANOVA and Tukey HSD) via the custom viime R package whose functions accept a CSV path and return a data frame) — https://github.com/girder/viime
- **OpenCPU** (Internal REST service providing the R-python integration so the Flask backend can invoke R statistical functions)
- **Pandas** (Parses the raw CSV and exchanges data frames with R through viime/opencpu.py)
- **Flask** (RESTful API backend that drives the statistical testing workflow)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('concentrations.csv'); from viime.opencpu import run; res = run('viime::anovaTukey', 'concentrations.csv'); res.to_csv('anova_tukey_results.csv')"
```

## Evaluation signals

- The output file anova_tukey_results.csv exists and parses as a table
- Every tested metabolite has an ANOVA p-value entry plus one Tukey p-value for each pairwise group comparison
- All reported p-values fall within [0, 1]
- R functions called via the OpenCPU bridge return valid data frames when given the CSV path (the viime package contract: 'accept a path to a CSV file as an argument and return a data frame')
- The number of Tukey p-value columns equals the number of unique pairwise group combinations (k choose 2 for k groups)

## Limitations

- The R processing functions run in a Dockerized OpenCPU instance (port 8004 in the README example); if that service is down, ANOVA/Tukey calls through viime/opencpu.py fail
- The OpenCPU bridge is described as work-in-progress backend code, so the pandas-to-R data frame exchange may not cover all cases
- Adding new R methods exposed to the API server requires adding the function to the viime R package and rebuilding the docker image
- The article does not state a multiple-testing correction applied to the Tukey p-values; significance selection is left to interactive check boxes

## Evidence

- [abstract] ANOVA with automatic Tukey: "Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively."
- [abstract] significance selection: "Metabolites that are significantly different in each of the intergroup comparisons can be selected for further analysis with a check box at the top of each column."
- [abstract] R-Python bridge: "The R-python integration is provided by a secondary REST service exposed internally via OpenCPU."
- [readme] R package contract: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [readme] bridge implementation: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames."
