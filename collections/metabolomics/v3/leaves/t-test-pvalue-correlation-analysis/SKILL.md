---
name: t-test-pvalue-correlation-analysis
description: Use when comparing imputation methods on MNAR-degraded targeted metabolomics
  data (FFA/BA matrices) where the criterion is preservation of two-group statistical
  sensitivity, not raw imputation accuracy.
when_to_use_negative:
- Missingness is MCAR/MAR (e.g. post-missing-retrieval untargeted GC/MS data) — the
  pipeline is designed for left-censored MNAR values caused by LOD/LOQ in targeted
  metabolomics
- The goal is raw imputation accuracy (NRMSE / observation distribution) rather than
  preservation of downstream group-difference statistics
- Data lack a two-group design matching the rep(c(0,1), each=80) label structure,
  since both t-test and supervised PLS-Procrustes metrics require the binary outcome
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3272
tools:
- name: R
  role: Host environment for the imputation evaluation pipeline (GSimp_evaluation.R
    / Imputation evaluations.R) that runs the t-test and Procrustes evaluations
  repo: https://github.com/WandeRum/GSimp
- name: GSimp
  role: One of the four MNAR imputation methods being compared; core GS_impute function
    with lo=-Inf, hi='min' for left-censored values
  repo: https://github.com/WandeRum/GSimp
- name: ropls
  role: Performs the supervised PLS-DA ordination (nPCs = 2) with group labels as
    outcome prior to Procrustes comparison
- name: vegan
  role: Computes the Procrustes sum of squared errors between original and imputed
    sample distributions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:d9f92d48bca135717a342d1a08a8d1821dc667e7e8a8ab24461b0c48854deb16
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# t-test-pvalue-correlation-analysis

## Summary

A labeled (supervised) evaluation that scores MNAR imputation methods (HM, QRILC, kNN-TN, GSimp) by how well they preserve group-difference signal: Pearson correlation of log-transformed two-sample t-test p-values (imputed vs original) plus PLS-DA Procrustes sum of squared errors between original and imputed sample ordinations.

## When to use

Use when comparing imputation methods on MNAR-degraded targeted metabolomics data (FFA/BA matrices) where the criterion is preservation of two-group statistical sensitivity, not raw imputation accuracy.

## When NOT to use

- Missingness is MCAR/MAR (e.g. post-missing-retrieval untargeted GC/MS data) — the pipeline is designed for left-censored MNAR values caused by LOD/LOQ in targeted metabolomics
- The goal is raw imputation accuracy (NRMSE / observation distribution) rather than preservation of downstream group-difference statistics
- Data lack a two-group design matching the rep(c(0,1), each=80) label structure, since both t-test and supervised PLS-Procrustes metrics require the binary outcome

## Inputs

- Complete original scaled metabolomics data matrix (FFA and BA targeted LC/MS datasets, e.g. real_data.csv)
- Two-class group labels vector: factor(rep(c(0, 1), each = 80))
- List of MNAR-degraded and imputed data matrices per method (HM, QRILC, kNN-TN, GSimp) generated at mis_var_prop = seq(.1, .8, .1), var_mis_prop = seq(.3, .6, .1)

## Outputs

- labeled_evaluation_metrics.csv with columns method, Miss_Num, t-test p-value Pearson correlation, and PLS-Procrustes SSE (one row per method × missing proportion)
- Per-method p-value correlation values between log-transformed imputed and original t-test p-values
- Per-method Procrustes sum of squared errors between original and imputed PLS-DA (nPCs = 2) sample ordinations

## How to apply

Load the complete original scaled data matrix and group labels defined as factor(rep(c(0, 1), each = 80)), together with the upstream list of MNAR-degraded/imputed datasets generated with mis_var_prop = seq(.1, .8, .1) and var_mis_prop = seq(.3, .6, .1) across HM, QRILC, kNN-TN, and GSimp. For each method × missing proportion, run per-variable two-sample Student's t-tests on imputed and original data with Ttest_cor_cal_plot(cor = 'P'), log-transform both p-value vectors, and compute Pearson's correlation restricted to the missing variables. In parallel, fit a supervised PLS ordination (ropls, nPCs = 2, group labels as outcome) via Procrustes_cal_plot(DR = 'PLS', outcome = group) and compute the vegan::procrustes sum of squared errors between original and imputed sample configurations. Assemble p-value correlations and PLS-Procrustes SSE into a tidy table (method, Miss_Num, both metrics) written to labeled_evaluation_metrics.csv; the expected trend is GSimp achieving the highest p-value correlation and lowest Procrustes SSE as missingness increases.

## Related tools

- **R** (Host environment for the imputation evaluation pipeline (GSimp_evaluation.R / Imputation evaluations.R) that runs the t-test and Procrustes evaluations) — https://github.com/WandeRum/GSimp
- **GSimp** (One of the four MNAR imputation methods being compared; core GS_impute function with lo=-Inf, hi='min' for left-censored values) — https://github.com/WandeRum/GSimp
- **ropls** (Performs the supervised PLS-DA ordination (nPCs = 2) with group labels as outcome prior to Procrustes comparison)
- **vegan** (Computes the Procrustes sum of squared errors between original and imputed sample distributions)

## Examples

```
cor(log(p_imp), log(p_orig), method = 'P')  # p-vectors from Ttest_cor_cal_plot(cor='P'); ordination via Procrustes_cal_plot(DR='PLS', outcome=group)
```

## Evaluation signals

- labeled_evaluation_metrics.csv contains exactly one row per method × missing proportion with both the p-value correlation and Procrustes SSE populated
- GSimp shows the highest log-transformed t-test p-value correlation and the lowest PLS-Procrustes SSE among HM, QRILC, kNN-TN, and GSimp as the number of missing variables increases
- p-value correlations are computed only over the missing variables, after log-transformation of both p-value vectors, using Pearson (cor = 'P')
- PLS ordination uses nPCs = 2 with the group factor as outcome, and Procrustes SSE is computed via vegan::procrustes between original and imputed configurations
- Missingness grid matches the upstream MNAR generation: mis_var_prop = seq(.1, .8, .1) and var_mis_prop = seq(.3, .6, .1)

## Limitations

- Evaluation is two-group (labels rep(c(0,1), each=80)) and FFA/BA-specific; extension to other designs or '-omics' scenarios is stated by the authors as untested and needing future evaluation
- Trends depend on the MNAR degradation parameters (mis_var_prop 0.1–0.8, var_mis_prop 0.3–0.6); different truncation points (e.g. 10% quantile vs known LOQ/LOD) are not fixed by the paper
- No quantitative runtime/convergence figures are given in the main text (deferred to supplementary S1 Table and S2 Fig), and no package versions are pinned for ropls/vegan/glmnet/imputeLCMD in the discussion

## Evidence

- [abstract] GSimp outperforms baseline methods on labeled metrics: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [intro] PLS-DA is the multivariate group-difference analysis: "For the multivariate analyses, we applied PLS-DA to distinguish the group differences"
- [intro] Comparison scope: GSimp vs HM, QRILC, kNN-TN on two real-world datasets: "we compared GSimp with HM, QRILC, and kNN-TN on two real-world metabolomics datasets and one simulation dataset"
- [readme] Evaluation functions live in GSimp_evaluation.R: "GSimp_evaluation.R contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
- [readme] MVI-evaluation repo hosts the evaluation pipeline scripts: [evidence span withheld — anchor status: external_doc]
