---
name: mnar-missing-data-simulation
description: Use when you have a complete (or near-complete) metabolomics data matrix,
  e.g. data_sim.csv or real_data.csv, and need to simulate left-censored MNAR missingness
  to evaluate imputation accuracy, distribution preservation, or statistical sensitivity.
when_to_use_negative:
- When the dataset's missingness is MCAR/MAR rather than left-censored MNAR — the
  paper states the majority of missing values in non-targeted GC/MS data were assumed
  MCAR/MAR before manual missing retrieval, and non-informative bounds (lo=-Inf, hi=Inf)
  apply there instead
- When the data already contain real missing values and no complete reference exists
  — degradation simulation requires a complete matrix to compute NRMSE, Procrustes
  SSE, and TPR against ground truth
- When evaluating right-censored missing values — use 'rsym' initialization with an
  informative lower truncation point rather than the left-censored cut-off mechanism
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: GSimp
  role: Source of MNAR_gen_imp, GS_impute, and evaluation functions (GSimp_evaluation.R)
    used to generate MNAR-degraded data and impute it
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Runtime environment for the simulation, imputation wrappers, and evaluation
    pipeline
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: Baseline imputer (impute.QRILC from imputeLCMD) applied to each MNAR-degraded
    matrix and used as GSimp initialization
- name: kNN-TN
  role: Truncated-normal kNN baseline (imputeKNN, k=3, distance='truncation', perc=0)
    applied to degraded matrices
  repo: https://github.com/WandeRum/GSimp
- name: vegan
  role: Procrustes SSE computation on PCA ordinations (2 PCs) comparing imputed vs.
    original observation distributions
- name: glmnet
  role: Elastic-net prediction model embedded in GS_impute's Gibbs sampler (imp_model='glmnet_pred')
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_007
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:f3f136fafc8cab04b890468b84398a06191d3eb15cd8546694bf5d7551ccb3e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - R
  - glmnet
  - imputeLCMD
  - kNN-TN
  - vegan
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mnar-missing-data-simulation

## Summary

Generates MNAR-degraded versions of complete metabolomics matrices using MNAR_gen_imp, introducing left-censored missingness at controlled proportions of missing variables and per-variable cut-offs, to benchmark imputation methods (GSimp, QRILC, kNN-TN, HM) against a known complete reference.

## When to use

Use when you have a complete (or near-complete) metabolomics data matrix, e.g. data_sim.csv or real_data.csv, and need to simulate left-censored MNAR missingness to evaluate imputation accuracy, distribution preservation, or statistical sensitivity.

## When NOT to use

- When the dataset's missingness is MCAR/MAR rather than left-censored MNAR — the paper states the majority of missing values in non-targeted GC/MS data were assumed MCAR/MAR before manual missing retrieval, and non-informative bounds (lo=-Inf, hi=Inf) apply there instead
- When the data already contain real missing values and no complete reference exists — degradation simulation requires a complete matrix to compute NRMSE, Procrustes SSE, and TPR against ground truth
- When evaluating right-censored missing values — use 'rsym' initialization with an informative lower truncation point rather than the left-censored cut-off mechanism

## Inputs

- Complete or scaled metabolomics data matrix (e.g. data_sim.csv, 160 samples x 76 variables; or real_data.csv containing FFAs and BAs) as an R data frame
- Mis_var_prop sequence of missing-variable proportions, e.g. seq(.1, .8, .1)
- Var_mis_prop range for per-variable missing fractions / MNAR cut-off draws, e.g. seq(.3, .6, .1)

## Outputs

- A series of MNAR-degraded data matrices at each missing-variable proportion, with known missing positions
- NA position indices (which(is.na(...), arr.ind=T)) defining the missing cells for NRMSE and TPR evaluation
- Imputed matrices from each method (GSimp, QRILC, kNN-TN, HM) on identical missingness patterns

## How to apply

Load the complete reference matrix (data_sim.csv, 160 samples x 76 variables, scaled, or real_data.csv with FFAs and BAs after log-transformation, centralization and scaling), then call MNAR_gen_imp with mis_var_prop=seq(.1, .8, .1) to vary the proportion of variables carrying missing values across eight levels, and var_mis_prop=seq(.3, .6, .1) so each variable's missing fraction draws its MNAR truncation cut-off from U(0.3, 0.6). This mimics LOD/LOQ-driven left-censoring characteristic of MS-based targeted metabolomics: values below a variable-specific cut-off are removed, producing degraded matrices whose ground truth is known. Each degraded matrix is then passed to imputation wrappers (sim_GS_wrapper with GS_impute iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min'; sim_QRILC_wrapper via impute.QRILC; sim_trKNN_wrapper via imputeKNN k=3, distance='truncation', perc=0; sim_HM_wrapper using half the variable minimum). Because the original complete data serve as reference, per-variable NRMSE on the missing positions, Procrustes SSE on PCA ordinations (2 PCs, vegan), and TPR of differential-variable detection (p_cut=0.05 and 0.01) can all be computed exactly. Use cores=1 for reproducible single-threaded generation, or parallel cores for large-scale runs.

## Related tools

- **GSimp** (Source of MNAR_gen_imp, GS_impute, and evaluation functions (GSimp_evaluation.R) used to generate MNAR-degraded data and impute it) — https://github.com/WandeRum/GSimp
- **R** (Runtime environment for the simulation, imputation wrappers, and evaluation pipeline) — https://github.com/WandeRum/GSimp
- **QRILC** (Baseline imputer (impute.QRILC from imputeLCMD) applied to each MNAR-degraded matrix and used as GSimp initialization)
- **kNN-TN** (Truncated-normal kNN baseline (imputeKNN, k=3, distance='truncation', perc=0) applied to degraded matrices) — https://github.com/WandeRum/GSimp
- **vegan** (Procrustes SSE computation on PCA ordinations (2 PCs) comparing imputed vs. original observation distributions)
- **glmnet** (Elastic-net prediction model embedded in GS_impute's Gibbs sampler (imp_model='glmnet_pred'))

## Examples

```
sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1), var_mis_prop=seq(.3, .6, .1), cores=1)
```

## Evaluation signals

- The degraded matrix must contain missingness in the intended fraction of variables at each mis_var_prop level (0.1–0.8) with per-variable missing fractions within the drawn U(0.3, 0.6) cut-offs
- Per-variable NRMSE computed only on the injected missing positions must show GSimp attaining the lowest Sum of Ranks (SOR) across the missing-variable range in FFA and BA datasets
- Procrustes SSE between original and imputed PCA ordinations (2 PCs) should be lowest for GSimp as missingness increases
- TPR = |S intersect S'| / |S| at p_cut=0.05 and 0.01 should be highest for GSimp across Miss_Num levels
- The complete reference matrix must remain unchanged so imputed values can be compared element-wise at the exact NA positions

## Limitations

- MNAR cut-offs are drawn from U(0.3, 0.6) and simulation parameters (e.g. N(0, 0.5^2) variable means, covariance from the diabetes dataset P=76 via SVD) are not fully justified in the paper, so results may not generalize to all metabolomics platforms
- The simulation targets left-censored MNAR from LOD/LOQ effects in targeted LC/MS data; it does not represent MCAR/MAR or right-censored mechanisms
- Reduced iteration counts (iters_each=50, iters_all=10) affect accuracy less but large-scale datasets require the parallel version with iters_all=20, iters_each=100 recommended for MCMC convergence

## Evidence

- [methods] MNAR_gen_imp call: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [methods] GS_impute defaults: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [abstract] MNAR motivation: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies"
- [readme] data_sim.csv description: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
- [readme] imputation evaluation pipeline: "**GSimp_evaluation.R** contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
