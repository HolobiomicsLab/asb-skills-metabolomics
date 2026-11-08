---
name: mnar-missing-data-evaluation
description: Use when complete (pre-missingness) metabolomics data plus group labels
  are available and several MNAR imputation methods must be ranked by how well they
  preserve univariate and supervised multivariate group-difference structure.
when_to_use_negative:
- Data with genuine missing values and no complete reference matrix — the evaluation
  requires an original complete data matrix as ground truth.
- Unsupervised evaluation goals (e.g., PCA-based structure preservation) — this skill
  is specifically group-label-informed (t-tests and PLS-DA supervised ordination).
- Missing values in non-targeted GC/MS data assumed to be MCAR/MAR before missing
  retrieval — GSimp's left-censored defaults (lo=-Inf, hi='min') target MNAR, and
  non-informative bounds should be used instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Gibbs sampler based left-censored imputation method being evaluated; also
    supplies MNAR_gen_imp for generating MNAR-degraded matrices and the Ttest_cor_cal_plot
    / Procrustes_cal_plot evaluation functions in GSimp_evaluation.R
  repo: https://github.com/WandeRum/GSimp
- name: ropls
  role: Performs supervised PLS ordination (PLS-DA, nPCs=2) with group labels as outcome
    for Procrustes comparison
  repo: https://github.com/WandeRum/GSimp
- name: vegan
  role: Computes the Procrustes sum of squared errors between original and imputed
    sample configurations
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Execution environment for the evaluation pipeline (GSimp_evaluation.R, Impute_wrapper.R,
    MVI_global.R)
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:35f63b0099e3e75f62f7dad9b39ad6885e5450b5aefb54642bb31db76f6d286c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - PLS-DA
  - R
  - ggplot2
  - ropls
  - vegan
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# mnar-missing-data-evaluation

## Summary

A group-label-informed evaluation pipeline for left-censored (MNAR) missing value imputation methods in targeted metabolomics, comparing HM, QRILC, kNN-TN, and GSimp by t-test p-value correlation and PLS-Procrustes concordance against the original complete data.

## When to use

Use when complete (pre-missingness) metabolomics data plus group labels are available and several MNAR imputation methods must be ranked by how well they preserve univariate and supervised multivariate group-difference structure.

## When NOT to use

- Data with genuine missing values and no complete reference matrix — the evaluation requires an original complete data matrix as ground truth.
- Unsupervised evaluation goals (e.g., PCA-based structure preservation) — this skill is specifically group-label-informed (t-tests and PLS-DA supervised ordination).
- Missing values in non-targeted GC/MS data assumed to be MCAR/MAR before missing retrieval — GSimp's left-censored defaults (lo=-Inf, hi='min') target MNAR, and non-informative bounds should be used instead.

## Inputs

- Complete scaled data matrix CSV (e.g., data_sim.csv, real_data.csv, targeted_data.csv, untargeted_data.csv)
- Group label vector (e.g., factor of rep(c(0,1), each=80))
- List of MNAR-degraded/imputed data matrices per method across missing proportions (mis_var_prop=seq(.1,.8,.1))
- R packages: ropls (PLS-DA), vegan (procrustes), ggplot2, imputeLCMD (QRILC), Trunc_KNN (kNN-TN), GSimp

## Outputs

- labeled_evaluation_metrics.csv with one row per method × missing proportion containing p-value correlation and PLS-Procrustes SSE
- Per-method comparison of log-transformed t-test p-value correlations vs. the original data
- PLS-Procrustes sum of squared errors between original and imputed sample ordinations (nPCs=2)

## How to apply

Generate a series of MNAR-degraded matrices from the complete scaled data (e.g., data_sim_sc) with MNAR_gen_imp using mis_var_prop=seq(.1,.8,.1) and per-variable quantile cutoffs drawn from U(0.3,0.6), then impute each with GSimp, QRILC, kNN-TN, and HM wrappers. For each method × missing proportion, run per-variable two-sample Student's t-tests on both imputed and original data (Ttest_cor_cal_plot with cor='P'), log-transform the p-value vectors, and compute Pearson correlation over the missing variables. In parallel, perform PLS-DA ordination with nPCs=2 using group labels rep(c(0,1), each=80) as the outcome via Procrustes_cal_plot(DR='PLS', outcome=group), and compute the Procrustes sum of squared errors between original and imputed sample configurations with vegan::procrustes. Assemble both metrics into a tidy table (method, Miss_Num, p-value correlation, PLS-Procrustes SSE) and write labeled_evaluation_metrics.csv; the method with the highest p-value correlation and lowest Procrustes SSE at increasing missingness is judged best.

## Related tools

- **GSimp** (Gibbs sampler based left-censored imputation method being evaluated; also supplies MNAR_gen_imp for generating MNAR-degraded matrices and the Ttest_cor_cal_plot / Procrustes_cal_plot evaluation functions in GSimp_evaluation.R) — https://github.com/WandeRum/GSimp
- **ropls** (Performs supervised PLS ordination (PLS-DA, nPCs=2) with group labels as outcome for Procrustes comparison) — https://github.com/WandeRum/GSimp
- **vegan** (Computes the Procrustes sum of squared errors between original and imputed sample configurations) — https://github.com/WandeRum/GSimp
- **R** (Execution environment for the evaluation pipeline (GSimp_evaluation.R, Impute_wrapper.R, MVI_global.R)) — https://github.com/WandeRum/GSimp

## Examples

```
source('GSimp_evaluation.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1))
```

## Evaluation signals

- labeled_evaluation_metrics.csv must contain exactly one row per method × missing proportion with both the p-value correlation and PLS-Procrustes SSE populated.
- The expected trend is that GSimp achieves the highest correlation of log-transformed t-test p-values and the lowest PLS-Procrustes SSE among HM, QRILC, kNN-TN, and GSimp as missingness increases in both FFA and BA datasets.
- Each generated MNAR matrix must have a missing-variable proportion matching the requested mis_var_prop level, and no values above the per-variable quantile cutoffs removed.
- PLS ordination must use nPCs=2 with the group labels as the outcome, and Procrustes SSE must be computed against the original data's ordination via vegan::procrustes.
- p-value vectors must be log-transformed before Pearson correlation (cor='P') is computed over the missing variables only.

## Limitations

- The evaluation was tuned with simulation-dataset parameters (U(0.3,0.6) quantile cutoffs, mis_var_prop 0.1–0.8 step 0.1); real-world datasets used 0.1–0.6 step 0.05 and U(0.1,0.5), so parameters must be matched to the dataset.
- Authors note GSimp should be evaluated in other complex scenarios ('other -omics') in the future, so extrapolation beyond targeted metabolomics is untested.
- Convergence depends on sufficient Gibbs sampler iterations (iters_all=20, iters_each=100 recommended); fewer iterations may affect imputation quality feeding the evaluation.
- No quantitative runtime or convergence figures are provided in the discussion for the parallel version (deferred to supplementary S1 Table and S2 Fig).

## Evidence

- [other] GSimp performs best on the group-label-informed evaluation: [evidence span withheld — anchor status: not_found]
- [other] PLS-Procrustes metric: [evidence span withheld — anchor status: not_found]
- [other] MNAR generation mechanism: [evidence span withheld — anchor status: not_found]
- [abstract] Evaluation pipeline comparison scope: "We compared GSimp with other three imputation methods on two real-world targeted metabolomics datasets and one simulation dataset using our imputation evaluation pipeline."
- [abstract] Univariate and multivariate analysis preservation: "GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [intro] PLS-DA for group differences: "For the multivariate analyses, we applied PLS-DA to distinguish the group differences"
- [methods] MNAR generation function: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [readme] Evaluation pipeline repo: [evidence span withheld — anchor status: not_found]
