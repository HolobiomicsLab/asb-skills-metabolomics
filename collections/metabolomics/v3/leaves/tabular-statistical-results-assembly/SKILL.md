---
name: tabular-statistical-results-assembly
description: Use when you have a preprocessed metabolomics concentration table (CSV
  with samples as rows, metabolite concentration columns, a group/factor column) with
  two or more sample groups and you need per-metabolite ANOVA and pairwise Tukey p-values
  assembled into one results table.
when_to_use_negative:
- The dataset has fewer than two groups — Viime restricts multivariate ANOVA to data
  with two or more groups.
- Input is not a preprocessed concentration table (e.g. raw platform output or spectra
  that have not been through Viime's upload, error correction, and imputation steps).
- The goal is a non-parametric group comparison — Viime's univariate option is the
  Wilcoxon rank sum test, not ANOVA.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3570
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: Web-based application whose backend runs the multivariate ANOVA and automatic
    post-hoc Tukey test over the uploaded concentration table and assembles per-metabolite
    p-value results.
  repo: https://github.com/girder/viime
- name: R
  role: Statistics stack invoked for the ANOVA/Tukey computation; R functions live
    in the devops/viime R package exposed to the API server.
  repo: https://github.com/girder/viime
- name: OpenCPU
  role: Secondary REST service that mediates R–Python integration so pandas data frames
    can be processed by the R statistical functions.
  repo: https://github.com/girder/viime
- name: pandas
  role: Parses the raw CSV concentration table and handles raw file parsing and normalization
    before statistical testing.
- name: Flask
  role: RESTful API backend that receives the parsed table and dispatches statistical
    processing requests.
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:ae7fc0db579a67228350b1705250583f09cb601f16545de169896be266441ac8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - OpenCPU
  - R
  - Viime
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3570
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tabular-statistical-results-assembly

## Summary

Assembles a metabolite-keyed results table from group-comparison statistics (ANOVA plus post-hoc Tukey HSD p-values for every pairwise inter-group comparison), produced by Viime's R-backed statistical analysis pipeline on a preprocessed concentration CSV.

## When to use

Use when you have a preprocessed metabolomics concentration table (CSV with samples as rows, metabolite concentration columns, a group/factor column) with two or more sample groups and you need per-metabolite ANOVA and pairwise Tukey p-values assembled into one results table.

## When NOT to use

- The dataset has fewer than two groups — Viime restricts multivariate ANOVA to data with two or more groups.
- Input is not a preprocessed concentration table (e.g. raw platform output or spectra that have not been through Viime's upload, error correction, and imputation steps).
- The goal is a non-parametric group comparison — Viime's univariate option is the Wilcoxon rank sum test, not ANOVA.

## Inputs

- Preprocessed metabolomics concentration table (CSV; samples as rows, metabolite concentration columns, one group/factor column)
- Group/factor column designating two or more sample groups
- Optional: per-metabolite significance selections from prior univariate screening (check box selections on the statistical comparison page)

## Outputs

- Results table keyed by metabolite with one ANOVA p-value and one Tukey HSD p-value per pairwise inter-group comparison
- anova_tukey_results.csv (or equivalent per-metabolite statistical results CSV)

## How to apply

Load the concentration table with pandas, ensuring one column is designated as the group/factor column (Viime's upload UI designates columns as primary ID, factor, group, or metabolite concentration). Split each metabolite concentration column by group so it can be tested across the two or more groups. For each metabolite, run a one-way ANOVA via the R statistics stack — Viime's backend calls R packages through OpenCPU, with viime/opencpu.py bridging pandas and R data frames. Automatically apply the post-hoc Tukey HSD test to compute pairwise p-values for each inter-group comparison of every tested metabolite. Assemble a results table keyed by metabolite with one ANOVA p-value column plus one Tukey p-value column per pairwise group comparison, and write it out (e.g. anova_tukey_results.csv). The rationale for running this through Viime's stack rather than ad-hoc scripts is that the R processing functions are packaged in the devops/viime R package, which is rebuilt whenever new methods are exposed to the API server, keeping analysis reproducible.

## Related tools

- **Viime** (Web-based application whose backend runs the multivariate ANOVA and automatic post-hoc Tukey test over the uploaded concentration table and assembles per-metabolite p-value results.) — https://github.com/girder/viime
- **R** (Statistics stack invoked for the ANOVA/Tukey computation; R functions live in the devops/viime R package exposed to the API server.) — https://github.com/girder/viime
- **OpenCPU** (Secondary REST service that mediates R–Python integration so pandas data frames can be processed by the R statistical functions.) — https://github.com/girder/viime
- **pandas** (Parses the raw CSV concentration table and handles raw file parsing and normalization before statistical testing.)
- **Flask** (RESTful API backend that receives the parsed table and dispatches statistical processing requests.) — https://github.com/girder/viime

## Examples

```
import pandas as pd; df = pd.read_csv('concentrations.csv'); groups = df['group']; anova_p, tukey_p = run_anova_tukey(df.drop(columns='group'), groups); anova_tukey_results.csv
```

## Evaluation signals

- Output CSV exists, parses as a table, and is keyed by metabolite.
- Contains one p-value entry for the ANOVA and one Tukey p-value for each pairwise inter-group comparison of every tested metabolite.
- All reported p-values fall within [0, 1].
- Number of Tukey columns equals the number of distinct pairwise group combinations for the groups present in the factor column.
- Every metabolite concentration column that passed error checks (numeric data, non-missing primary ID) appears as a row in the results table.

## Limitations

- Multivariate ANOVA is only applicable to datasets with two or more groups; single-group data cannot use this skill.
- The R processing functions exposed via OpenCPU require rebuilding the docker image when new functions are added — results depend on the packaged viime R package version.
- Viime's workflow assumes input data have already been through preprocessing and annotation; it picks up the workflow after preprocessing and annotation.
- Errors detected during upload (missing data above threshold, non-numeric concentration data, missing or non-unique primary ID, low-variance metabolites) will block or degrade the statistical results if not corrected first.

## Evidence

- [abstract] Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively.: "Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively."
- [abstract] automatic post-hoc Tukey test: [evidence span withheld — anchor status: not_found]
- [readme] OpenCPU as the R–Python integration layer: "The R-python integration is provided by a secondary REST service exposed internally via OpenCPU."
- [readme] devops/viime R package and CSV processing contract: [evidence span withheld — anchor status: not_found]
- [abstract] preprocessing/annotation as precondition: "The intent of Viime package is to pick up the workflow after preprocessing and annotation and go all the way through statistical analysis and visualization."
