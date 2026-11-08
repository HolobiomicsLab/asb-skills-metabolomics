---
name: pls-supervised-ordination
description: Use when comparing imputation methods (e.g., GSimp vs HM, QRILC, kNN-TN)
  on MNAR-degraded FFA/BA metabolomics data and you need a supervised, group-label-informed
  multivariate metric of imputation fidelity alongside univariate t-test p-value correlations.
when_to_use_negative:
- The data have no group labels or outcome variable — PLS-DA is supervised and cannot
  be fit without the outcome, so unsupervised PCA ordination is the appropriate alternative
- The dominant missingness mechanism is MCAR/MAR (e.g., untargeted GC/MS data before
  manual missing retrieval), where the paper's left-censored MNAR evaluation framing
  does not apply
- The goal is per-variable imputation accuracy or distribution checks rather than
  preservation of between-group multivariate structure
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: ropls
  role: Performs the supervised PLS-DA ordination with nPCs=2 using group labels as
    the outcome variable
- name: vegan
  role: Fits the Procrustes rotation between original and imputed PLS sample distributions
    and yields the sum of squared errors metric
- name: GSimp_evaluation.R / MVI-evaluation
  role: Provides Procrustes_cal_plot and the imputation evaluation pipeline (missing
    generation, imputations, evaluations)
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:c3259a8c8c90aeaef4583700fd83acd428668d024bd027612d138fb99f1b9b0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/MVI-evaluation
  tools:
  - GSimp
  - GSimp_evaluation.R / MVI-evaluation
  - PLS-DA
  - R
  - ggplot2
  - ropls
  - vegan
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# pls-supervised-ordination

## Summary

A supervised multivariate evaluation step that performs PLS-DA ordination (ropls, nPCs=2) on original and imputed MNAR-degraded targeted metabolomics matrices using binary group labels, then quantifies agreement between sample distributions via vegan::procrustes sum of squared errors to judge how well an imputation method preserves group-difference information.

## When to use

Use when comparing imputation methods (e.g., GSimp vs HM, QRILC, kNN-TN) on MNAR-degraded FFA/BA metabolomics data and you need a supervised, group-label-informed multivariate metric of imputation fidelity alongside univariate t-test p-value correlations.

## When NOT to use

- The data have no group labels or outcome variable — PLS-DA is supervised and cannot be fit without the outcome, so unsupervised PCA ordination is the appropriate alternative
- The dominant missingness mechanism is MCAR/MAR (e.g., untargeted GC/MS data before manual missing retrieval), where the paper's left-censored MNAR evaluation framing does not apply
- The goal is per-variable imputation accuracy or distribution checks rather than preservation of between-group multivariate structure

## Inputs

- Complete original scaled data matrix (e.g., real_data.csv containing FFA and BA variables, log-transformed and scale-normalized)
- Binary group label vector defined as factor(rep(c(0, 1), each=80))
- List of MNAR-degraded/imputed data matrices per method and per missingness level (mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1))

## Outputs

- PLS-Procrustes sum of squared errors (SSE) per method × missing-variable count (Miss_Num)
- labeled_evaluation_metrics.csv: tidy table with columns method, Miss_Num, t-test p-value correlation, and PLS-Procrustes SSE

## How to apply

Load the complete original scaled data matrix (e.g., real_data.csv containing FFAs and BAs after log-transformation, QRILC initialization, and centralization/scaling) together with the binary group labels defined as rep(c(0, 1), each=80) as a factor. For each imputation method and MNAR-degradation level (mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1)), run supervised PLS ordination on both the imputed and the original data using the ropls package with nPCs=2 and the group labels as the outcome, via Procrustes_cal_plot(DR='PLS', outcome=group). Then fit vegan::procrustes between the original and imputed sample score distributions and record the Procrustes sum of squared errors (SSE) as the disagreement metric. Assemble SSE values (paired with the Pearson correlation of log-transformed two-sample t-test p-values from Ttest_cor_cal_plot(cor='P')) into a tidy table keyed by method and Miss_Num, written to labeled_evaluation_metrics.csv. Rationale: PLS-DA is supervised, so it directly probes whether imputation preserved the group separation the paper used to distinguish sample classes; a lower Procrustes SSE means the imputed ordination geometry better matches the original complete-data ordination.

## Related tools

- **ropls** (Performs the supervised PLS-DA ordination with nPCs=2 using group labels as the outcome variable)
- **vegan** (Fits the Procrustes rotation between original and imputed PLS sample distributions and yields the sum of squared errors metric)
- **GSimp_evaluation.R / MVI-evaluation** (Provides Procrustes_cal_plot and the imputation evaluation pipeline (missing generation, imputations, evaluations)) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
Procrustes_cal_plot(DR='PLS', outcome=group)
```

## Evaluation signals

- labeled_evaluation_metrics.csv contains exactly one row per method × missing proportion with both the p-value correlation and PLS-Procrustes SSE populated
- GSimp (expected winner) shows the highest correlation of log-transformed t-test p-values and the lowest PLS-Procrustes SSE among HM, QRILC, kNN-TN, and GSimp at increasing Miss_Num in both FFA and BA datasets
- PLS ordination used nPCs=2 with the group factor rep(c(0, 1), each=80) as outcome — verify via Procrustes_cal_plot(DR='PLS', outcome=group) call
- Procrustes SSE is computed between the original (complete-data) ordination and each imputed-data ordination on the same samples, so SSE ≈ 0 indicates perfect geometric agreement
- The trend should hold or strengthen as the number of missing variables increases, since greater MNAR degradation amplifies differences between imputation methods

## Limitations

- Results are specific to the group-label-informed evaluation with the binary rep(c(0,1), each=80) design; multi-class or continuous outcomes were not tested in this pipeline
- No specific version numbers beyond ropls 1.8.0 and vegan 2.4-5 are pinned, and runtime/convergence figures are deferred to supplementary material (S1 Table, S2 Fig)
- The supervised PLS-DA metric complements but does not replace univariate checks; the paper evaluates sensitivity jointly via log-transformed t-test p-value correlation

## Evidence

- [intro] PLS-DA applied to distinguish group differences: "For the multivariate analyses, we applied PLS-DA to distinguish the group differences"
- [abstract] Evaluation pipeline context for GSimp vs other methods: "We compared GSimp with other three imputation methods on two real-world targeted metabolomics datasets and one simulation dataset using our imputation evaluation pipeline."
- [readme] Evaluation pipeline source location: [evidence span withheld — anchor status: not_found]
- [discussion] Method comparison scope: "We evaluated GSimp with other three imputation methods (kNN-TN, QRILC, and HM)"
