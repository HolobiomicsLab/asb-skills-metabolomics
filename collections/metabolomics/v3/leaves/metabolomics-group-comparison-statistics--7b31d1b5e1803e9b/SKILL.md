---
name: metabolomics-group-comparison-statistics
description: Use when a preprocessed metabolomics concentration table (CSV/XLSX, samples
  as rows, metabolite columns, a group/factor column) has two or more sample groups
  and you need per-metabolite significance plus pairwise inter-group p-values.
when_to_use_negative:
- The dataset has not yet passed Viime's upstream treatment pipeline (error correction,
  imputation of MCAR/MNAR blocks, normalization, log/square-root/cube-root transformation,
  autoscaling/Pareto/range/vast/level scaling) — run those steps first.
- You need univariate two-group testing only on data with a single group — Viime applies
  Wilcoxon and ANOVA on data with two or more groups.
- You expect significance-adjusted (e.g. FDR/Bonferroni) values rather than raw p-values
  — the article reports Wilcoxon rank sum and Tukey post-hoc p-values only.
edam_operation: http://edamontology.org/operation_0384
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: Viime
  role: web application exposing the group-comparison statistics workflow (Wilcoxon,
    ANOVA + Tukey) on uploaded metabolomics tables
  repo: https://github.com/girder/viime
- name: R
  role: hosts the specialized statistics (Wilcoxon rank sum, ANOVA, post-hoc Tukey)
    executed inside the OpenCPU service
- name: OpenCPU
  role: secondary REST service that bridges the Flask backend to the R processing
    functions
  repo: https://www.opencpu.org/
- name: Pandas
  role: parses raw metabolomics files and exchanges data frames with the R layer via
    viime/opencpu.py
- name: Flask
  role: RESTful backend API through which the statistics calls are issued
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:7917d072e21fd55f53a5662293ccc888135e7b711fb53dd5964519c52e203354
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
  edam_operation: http://edamontology.org/operation_0384
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-group-comparison-statistics

## Summary

Performs univariate and multivariate group-comparison statistics on metabolomics concentration tables: Wilcoxon rank sum tests per metabolite plus a multivariate ANOVA with an automatic post-hoc Tukey HSD test that yields pairwise p-values for every inter-group comparison.

## When to use

Use when a preprocessed metabolomics concentration table (CSV/XLSX, samples as rows, metabolite columns, a group/factor column) has two or more sample groups and you need per-metabolite significance plus pairwise inter-group p-values.

## When NOT to use

- The dataset has not yet passed Viime's upstream treatment pipeline (error correction, imputation of MCAR/MNAR blocks, normalization, log/square-root/cube-root transformation, autoscaling/Pareto/range/vast/level scaling) — run those steps first.
- You need univariate two-group testing only on data with a single group — Viime applies Wilcoxon and ANOVA on data with two or more groups.
- You expect significance-adjusted (e.g. FDR/Bonferroni) values rather than raw p-values — the article reports Wilcoxon rank sum and Tukey post-hoc p-values only.

## Inputs

- preprocessed metabolomics concentration table (CSV or .xlsx) with samples as rows, metabolite concentration columns, and a group/factor column
- CSV file path accepted by the viime R package functions exposed via OpenCPU

## Outputs

- per-metabolite ANOVA p-values
- per-metabolite pairwise Tukey HSD p-value table (one column per inter-group comparison)
- Wilcoxon rank sum test p-values for two-group comparisons
- set of significantly altered metabolites selected for further analysis

## How to apply

Load the error-corrected, normalized, transformed and scaled concentration table (CSV with samples as rows, metabolite concentration columns, and a designated group/factor column) into pandas. For datasets with two groups, apply the univariate Wilcoxon rank sum test per metabolite; for two or more groups, run the multivariate ANOVA across groups using the R statistics stack reached through the viime OpenCPU bridge (viime/opencpu.py, which shuttles pandas data frames to and from R). For each metabolite with two or more groups, automatically follow the ANOVA with a post-hoc Tukey HSD test that computes a p-value for every pairwise inter-group comparison. Assemble a results table keyed by metabolite with the ANOVA p-value and one p-value column per pairwise Tukey comparison. Significantly different metabolites in each intergroup comparison can then be flagged via a check box at the top of each column for downstream analysis (volcano plots, heatmaps, networks).

## Related tools

- **Viime** (web application exposing the group-comparison statistics workflow (Wilcoxon, ANOVA + Tukey) on uploaded metabolomics tables) — https://github.com/girder/viime
- **R** (hosts the specialized statistics (Wilcoxon rank sum, ANOVA, post-hoc Tukey) executed inside the OpenCPU service)
- **OpenCPU** (secondary REST service that bridges the Flask backend to the R processing functions) — https://www.opencpu.org/
- **Pandas** (parses raw metabolomics files and exchanges data frames with the R layer via viime/opencpu.py)
- **Flask** (RESTful backend API through which the statistics calls are issued)

## Examples

```
cd devops && docker build -t viime . && docker run --rm --name viime -p 8004:8004 viime   # then call the viime R ANOVA/Tukey functions on a CSV via OpenCPU from flask
```

## Evaluation signals

- Output p-value table exists, parses as a table, and contains a p-value entry for the ANOVA and one for every pairwise Tukey comparison of each tested metabolite
- All reported p-values fall within [0, 1]
- Number of Tukey p-value columns matches the pairwise count of groups defined in the group/factor column (k*(k-1)/2 for k groups)
- For two-group datasets, Wilcoxon rank sum results are produced instead of (or alongside) ANOVA
- Metabolites flagged significant in each intergroup comparison can be selected via the column-top check box and propagate to downstream volcano/heatmap/network analyses

## Limitations

- Multivariate ANOVA and automatic Tukey testing are only available for datasets with two or more groups; single-group data cannot be tested
- R processing requires a separately built OpenCPU docker image (cd devops; docker build -t viime .) — changes to exposed functions require rebuilding the image
- The OpenCPU bridge code (viime/opencpu.py) is described as work-in-progress
- Annotation is not required but the skill picks up the workflow only after preprocessing

## Evidence

- [abstract] ANOVA with post-hoc Tukey: "Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively."
- [abstract] post-hoc Tukey test detail: [evidence span withheld — anchor status: not_found]
- [abstract] tool stack for the statistics: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [readme] R-python bridge mechanics: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames."
- [readme] OpenCPU service setup: [evidence span withheld — anchor status: not_found]
- [abstract] significance selection: "Metabolites that are significantly different in each of the intergroup comparisons can be selected for further analysis with a check box at the top of each column."
