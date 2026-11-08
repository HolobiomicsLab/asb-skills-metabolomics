---
name: rank-based-method-comparison
description: Use when several imputation/substitution methods (e.g., GSimp, HM, QRILC,
  kNN-TN) must be compared fairly on left-censored MNAR missing values across varying
  numbers of missing variables, and NRMSE values alone need aggregation into a single
  comparable metric.
when_to_use_negative:
- The missing values are MCAR/MAR rather than left-censored MNAR (e.g., post-missing-retrieval
  non-targeted GC/MS data assumed mostly MCAR/MAR), where rank-based MNAR-accuracy
  comparison on left-censored degradation is not the target evaluation
- Only a single imputation method is being evaluated and no cross-method ranking is
  needed
- The complete (ground-truth) reference matrix is unavailable, since per-variable
  NRMSE on missing positions cannot be computed
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: GSimp
  role: Competing imputation method evaluated (GS_impute with iters_each=50, iters_all=10,
    initial='qrilc', lo=-Inf, hi='min', imp_model='glmnet_pred'); also supplies MNAR_gen_imp
    and NRMSE_rank_cal_plot evaluation functions in GSimp_evaluation.R
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: Competing imputation method (impute.QRILC wrapper) ranked against GSimp in
    the SOR comparison
- name: kNN-TN
  role: Competing imputation method (imputeKNN with k=3, distance='truncation', perc=0
    via sim_trKNN_wrapper) ranked against GSimp in the SOR comparison
  repo: https://github.com/WandeRum/GSimp
- name: HM
  role: Half-minimum substitution baseline (sim_HM_wrapper) ranked against GSimp in
    the SOR comparison
- name: R
  role: Execution environment for the evaluation pipeline (MVI_global.R, Impute_wrapper.R,
    Imputation evaluations.R)
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:aa9b125b2d237600ea79d3e766098f7aa823679f083eb5189ad7fd9cb72cecfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - HM
  - QRILC
  - R
  - kNN-TN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# rank-based-method-comparison

## Summary

Ranks competing imputation methods by per-variable NRMSE at each missing-variable level and aggregates the ranks into a Sum of Ranks (SOR), enabling a distribution-free comparison of methods on MNAR-degraded targeted metabolomics data (FFA and BA datasets).

## When to use

Use when several imputation/substitution methods (e.g., GSimp, HM, QRILC, kNN-TN) must be compared fairly on left-censored MNAR missing values across varying numbers of missing variables, and NRMSE values alone need aggregation into a single comparable metric.

## When NOT to use

- The missing values are MCAR/MAR rather than left-censored MNAR (e.g., post-missing-retrieval non-targeted GC/MS data assumed mostly MCAR/MAR), where rank-based MNAR-accuracy comparison on left-censored degradation is not the target evaluation
- Only a single imputation method is being evaluated and no cross-method ranking is needed
- The complete (ground-truth) reference matrix is unavailable, since per-variable NRMSE on missing positions cannot be computed

## Inputs

- Complete reference data matrix (e.g., real_data.csv targeted metabolomics containing FFAs and BAs, log-transformed, centralized and scaled)
- MNAR-degraded versions of the matrix generated at mis_var_prop=seq(.1,.8,.1) with var_mis_prop=seq(.3,.6,.1)
- Imputed matrices from each competing method (GSimp, QRILC, kNN-TN, HM)

## Outputs

- Per-variable NRMSE values at missing positions for each method and missing-variable level
- Method ranks at each missing-variable level
- SOR (Sum of Ranks) table per method per missing proportion (e.g., sor_results.csv)

## How to apply

Generate a series of MNAR-degraded datasets (e.g., mis_var_prop=seq(.1,.8,.1), per-variable MNAR cut-offs from var_mis_prop=seq(.3,.6,.1)) from a complete reference matrix. Impute each degraded dataset with every competing wrapper (GS_impute with iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min', imp_model='glmnet_pred'; impute.QRILC; kNN-TN with k=3, distance='truncation', perc=0; and half-minimum substitution). For each method and each missing-variable level, compute per-variable NRMSE on the missing positions against the complete reference. Rank the methods at each level and aggregate the ranks into the NRMSE-based Sum of Ranks (SOR) via NRMSE_rank_cal_plot, writing per-method SOR values per missing proportion (e.g., sor_results.csv). The method with the lowest SOR across the missing-variable range is the most accurate imputer.

## Related tools

- **GSimp** (Competing imputation method evaluated (GS_impute with iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min', imp_model='glmnet_pred'); also supplies MNAR_gen_imp and NRMSE_rank_cal_plot evaluation functions in GSimp_evaluation.R) — https://github.com/WandeRum/GSimp
- **QRILC** (Competing imputation method (impute.QRILC wrapper) ranked against GSimp in the SOR comparison)
- **kNN-TN** (Competing imputation method (imputeKNN with k=3, distance='truncation', perc=0 via sim_trKNN_wrapper) ranked against GSimp in the SOR comparison) — https://github.com/WandeRum/GSimp
- **HM** (Half-minimum substitution baseline (sim_HM_wrapper) ranked against GSimp in the SOR comparison)
- **R** (Execution environment for the evaluation pipeline (MVI_global.R, Impute_wrapper.R, Imputation evaluations.R)) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
R: sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1)); then rank methods per level with NRMSE_rank_cal_plot and write sor_results.csv
```

## Evaluation signals

- The SOR table should show GSimp attaining the lowest SOR across the full missing-variable range (mis_var_prop 0.1–0.8) on both FFA and BA datasets, consistent with the paper's finding
- Per-method SOR values must be monotonically interpretable aggregates of the per-level ranks — verify each SOR equals the sum of method ranks across missing-variable levels
- NRMSE must be computed only on the missing positions between each imputed matrix and the complete reference, not on observed positions
- Rank ties or ordering at each level should be reproducible when the evaluation pipeline (Imputation evaluations.R) is re-run with the same MNAR generation parameters
- Output file sor_results.csv should contain one SOR row per method per missing proportion

## Limitations

- Ranking depends on the specific MNAR degradation parameters (mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1)); results may not generalize to other missingness regimes or other -omics data
- The comparison is only as fair as the wrapper parameterizations; e.g., kNN-TN parameters (k=3, truncation distance) and HM substitution were fixed as in the paper
- NRMSE on missing positions requires a complete reference matrix, so this evaluation design applies only to simulated/degraded missingness, not to truly unknown missing patterns in raw data

## Evidence

- [methods] Rank the four methods at each missing-variable level and aggregate ranks into the NRMSE-based Sum of Ranks (SOR): [evidence span withheld — anchor status: not_found]
- [abstract] GSimp had the best performance with the lowest Sum of Ranks: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [supplementary] SOR on simulation dataset along with different numbers of missing variables based on four different numbers of iterations: "SOR on simulation dataset along with different numbers of missing variables based on four different numbers of iterations"
- [readme] GS_impute is the core function for the imputation of missing data and tracing the Gibbs sampler: "The function *GS_impute* is the core function for the imputation of missing data and tracing the Gibbs sampler with certain missing positions."
- [readme] Imputation evaluations.R contains our imputation evaluation pipeline (missng generation, imputations, evaluations): [evidence span withheld — anchor status: external_doc]
