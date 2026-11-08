---
name: quantile-cutoff-variable-selection
description: Use when building an imputation evaluation benchmark that requires MNAR
  left-censored missingness injected into a scaled data matrix at controlled missing-proportion
  levels.
when_to_use_negative:
- Input data has not yet been centralized and scaled, so per-variable quantile cut-offs
  would be drawn on an inconsistent scale.
- Missingness should be MCAR/MAR rather than left-censored MNAR, since non-informative
  bounds (e.g., -Inf to +Inf) apply in that setting.
edam_operation: http://edamontology.org/operation_3555
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: GSimp
  role: provides the MNAR_gen_imp function and evaluation pipeline for generating
    MNAR left-censored missingness
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: runtime environment for executing MNAR_gen_imp and the evaluation pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:d6a0e941207c0991b8251f6aeb23fb73d119ae259ec439a2ba6dd255dfb44fa1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - R
  edam_operation: http://edamontology.org/operation_3555
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# quantile-cutoff-variable-selection

## Summary

Generates MNAR left-censored missingness by drawing a per-variable quantile cut-off and setting values below that cut-off to NA, producing a series of missing-proportion matrices used to evaluate imputation methods.

## When to use

Use when building an imputation evaluation benchmark that requires MNAR left-censored missingness injected into a scaled data matrix at controlled missing-proportion levels.

## When NOT to use

- Input data has not yet been centralized and scaled, so per-variable quantile cut-offs would be drawn on an inconsistent scale.
- Missingness should be MCAR/MAR rather than left-censored MNAR, since non-informative bounds (e.g., -Inf to +Inf) apply in that setting.

## Inputs

- complete scaled data matrix (e.g., data_sim_sc from data_sim.csv)
- vector of target missing proportions (e.g., seq(0.1, 0.6, 0.05) for real data; seq(0.1, 0.8, 0.1) for simulation)

## Outputs

- list of MNAR data matrices with NA entries below per-variable quantile cut-offs
- per-level MNAR matrices saved as CSV files in the output directory

## How to apply

Load the complete data matrix (e.g., real_data.csv or data_sim.csv) and scale it before injection. For each target missing-proportion level (seq(0.1, 0.6, 0.05) for real-world datasets or seq(0.1, 0.8, 0.1) for the simulation dataset), draw a per-variable quantile cut-off from U(0.1, 0.5) (or U(0.3, 0.6) for simulation). Then set to NA all elements whose values fall below that per-variable cut-off, and repeat for each proportion level, collecting the resulting MNAR matrices. Validate that each output matrix's missing-variable fraction matches the requested level and that no values above the cut-offs were removed.

## Related tools

- **GSimp** (provides the MNAR_gen_imp function and evaluation pipeline for generating MNAR left-censored missingness) — https://github.com/WandeRum/GSimp
- **R** (runtime environment for executing MNAR_gen_imp and the evaluation pipeline)

## Examples

```
Rscript -e "source('GSimp_evaluation.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1))"
```

## Evaluation signals

- For each output MNAR matrix, the proportion of missing variables matches the requested level.
- No data elements above the per-variable quantile cut-off were removed (only left-tail values are NA).
- The series of MNAR matrices spans the requested range of missing proportions (e.g., 0.1–0.6 step 0.05 for real-world data).
- Per-variable quantile cut-offs are drawn from the specified uniform distribution (U(0.1, 0.5) or U(0.3, 0.6)).

## Limitations

- Requires a complete (fully imputed or simulated) data matrix as the starting point, since missingness is injected.
- Per-variable cut-off distributions (U(0.1, 0.5) vs U(0.3, 0.6)) must match the dataset type; mismatched cut-off ranges change the injected MNAR pattern.
- The injected missingness is strictly left-censored (values below cut-off) and does not cover MCAR/MAR patterns.

## Evidence

- [methods] MNAR_gen_imp: [evidence span withheld — anchor status: not_found]
- [methods] quantile cut-off: [evidence span withheld — anchor status: not_found]
- [methods] missing proportion range: [evidence span withheld — anchor status: not_found]
- [methods] cut-off distribution: [evidence span withheld — anchor status: not_found]
- [readme] GSimp.R content: "GSimp_evaluation.R contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline"
- [methods] impute wrapper call: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1)"
- [methods] left-censored default: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values"
