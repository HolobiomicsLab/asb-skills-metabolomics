---
name: group-label-informed-metabolomics-statistics
description: Use when comparing missing-value imputation methods on metabolomics data
  with known two-group labels, and you need label-informed metrics (t-test p-value
  correlation, PLS-Procrustes SSE) rather than only reconstruction error.
when_to_use_negative:
- The missingness mechanism is MCAR/MAR (e.g., post-missing-retrieval untargeted GC/MS
  data) rather than MNAR left-censoring from LOD/LOQ — the paper's labeled evaluation
  targets MNAR-degraded targeted metabolomics data
- You have no group labels or outcome variable — the t-test and supervised PLS-DA
  evaluation both require two-group labels (rep(c(0,1), each=80))
- You only need raw imputation accuracy (e.g., SOR) without judging preservation of
  univariate or multivariate group-difference statistics
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0092
tools:
- name: GSimp (GSimp_evaluation.R)
  role: Provides MNAR generation and the evaluation functions Ttest_cor_cal_plot and
    Procrustes_cal_plot used to compute p-value correlations and PLS-Procrustes SSE
  repo: https://github.com/WandeRum/GSimp
- name: ropls
  role: Performs the supervised PLS-DA ordination (nPCs=2) with group labels as outcome
    prior to Procrustes comparison (version 1.8.0 in the pipeline)
- name: vegan
  role: Computes the Procrustes sum of squared errors between original and imputed
    sample distributions (version 2.4-5 in the pipeline)
- name: ggplot2
  role: Visualizes the assembled labeled evaluation metrics across methods and missing-variable
    counts (version 2.2.1.9000 in the pipeline)
- name: MVI-evaluation
  role: Repository containing the imputation evaluation pipeline (missing generation,
    imputations, evaluations) that hosts Imputation evaluations.R
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:6ed7670746b85478b97218ce34edcfa564eef19f66f2e57d3db9d3116744bde0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - GSimp (GSimp_evaluation.R)
  - MVI-evaluation
  - PLS-DA
  - R
  - ggplot2
  - ropls
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# group-label-informed-metabolomics-statistics

## Summary

A supervised evaluation framework that judges imputation quality by whether group-difference signal is preserved: per-variable two-sample t-test p-value correlation and PLS-DA ordination compared by Procrustes sum of squared errors between imputed and original data.

## When to use

Use when comparing missing-value imputation methods on metabolomics data with known two-group labels, and you need label-informed metrics (t-test p-value correlation, PLS-Procrustes SSE) rather than only reconstruction error.

## When NOT to use

- The missingness mechanism is MCAR/MAR (e.g., post-missing-retrieval untargeted GC/MS data) rather than MNAR left-censoring from LOD/LOQ — the paper's labeled evaluation targets MNAR-degraded targeted metabolomics data
- You have no group labels or outcome variable — the t-test and supervised PLS-DA evaluation both require two-group labels (rep(c(0,1), each=80))
- You only need raw imputation accuracy (e.g., SOR) without judging preservation of univariate or multivariate group-difference statistics

## Inputs

- Complete original scaled data matrix (e.g., real_data.csv-derived FFA/BA targeted metabolomics matrix with no missing values)
- Binary group labels defined as factor(rep(c(0, 1), each=80))
- List of MNAR-degraded and imputed data matrices from MNAR_gen_imp with mis_var_prop=seq(.1,.8,.1) and var_mis_prop=seq(.3,.6,.1)
- Vector of missing-variable counts (Miss_Num) per missing proportion

## Outputs

- labeled_evaluation_metrics.csv with one row per method x missing proportion and columns method, Miss_Num, t-test p-value Pearson correlation, and PLS-Procrustes SSE
- Per-metric comparison plots (ggplot2) of p-value correlation and Procrustes SSE across increasing missingness for HM, QRILC, kNN-TN, and GSimp

## How to apply

Load the complete original scaled data matrix and the group labels defined as rep(c(0, 1), each=80) as a factor, together with the upstream list of MNAR-degraded/imputed datasets generated with mis_var_prop=seq(.1,.8,.1) and var_mis_prop=seq(.3,.6,.1). For each imputation method (HM, QRILC, kNN-TN, GSimp) and missing proportion, run per-variable two-sample Student's t-tests on imputed and original data with Ttest_cor_cal_plot (cor='P'), log-transform the p-value vectors, and compute Pearson's correlation between them for the missing variables. In parallel, perform supervised PLS ordination with nPCs=2 using the group labels as outcome via ropls inside Procrustes_cal_plot(DR='PLS', outcome=group), then compute the Procrustes sum of squared errors between original and imputed sample distributions with vegan::procrustes. Assemble p-value correlations and PLS-Procrustes SSE into a tidy table with columns method, Miss_Num, and both metrics, written to labeled_evaluation_metrics.csv. The rationale is that an imputation method suited to MNAR left-censored missing values (caused by LOD/LOQ in MS-based targeted metabolomics) should best retain statistical sensitivity and multivariate group separation relative to the complete original data.

## Related tools

- **GSimp (GSimp_evaluation.R)** (Provides MNAR generation and the evaluation functions Ttest_cor_cal_plot and Procrustes_cal_plot used to compute p-value correlations and PLS-Procrustes SSE) — https://github.com/WandeRum/GSimp
- **ropls** (Performs the supervised PLS-DA ordination (nPCs=2) with group labels as outcome prior to Procrustes comparison (version 1.8.0 in the pipeline))
- **vegan** (Computes the Procrustes sum of squared errors between original and imputed sample distributions (version 2.4-5 in the pipeline))
- **ggplot2** (Visualizes the assembled labeled evaluation metrics across methods and missing-variable counts (version 2.2.1.9000 in the pipeline))
- **MVI-evaluation** (Repository containing the imputation evaluation pipeline (missing generation, imputations, evaluations) that hosts Imputation evaluations.R) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
source('GSimp_evaluation.R'); Ttest_cor_cal_plot(imp_data, orig_data, cor='P'); Procrustes_cal_plot(DR='PLS', outcome=group)
```

## Evaluation signals

- labeled_evaluation_metrics.csv contains exactly one row per method x missing proportion with both the p-value correlation and PLS-Procrustes SSE populated (no NAs)
- Trend check: GSimp achieves the highest log-transformed t-test p-value Pearson correlation among HM, QRILC, kNN-TN, and GSimp as missingness increases in both FFA and BA datasets
- Trend check: GSimp achieves the lowest PLS-Procrustes sum of squared errors among the four methods at varying numbers of missing variables
- Metric directionality sanity: p-value correlations fall and Procrustes SSE rises as the number of missing variables increases for all methods
- Reproducibility check: group labels are factor(rep(c(0,1), each=80)) and PLS uses nPCs=2 with DR='PLS' and the group labels as outcome

## Limitations

- Evaluation is limited to two real-world targeted metabolomics datasets (FFA and BA) and one simulation dataset; extension to other -omics or complex scenarios is untested and the authors state future evaluation is needed
- No specific version numbers reported for R packages glmnet, vegan, ropls, or imputeLCMD in the discussion, complicating exact reproduction
- The evaluation depends on upstream MNAR simulation parameter choices (e.g., mis_var_prop=seq(.1,.8,.1)) that are not fully justified, and results reflect MNAR left-censoring rather than MCAR/MAR missingness

## Evidence

- [abstract] Univariate and multivariate group-difference evaluation: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [intro] PLS-DA supervised ordination basis: "For the multivariate analyses, we applied PLS-DA to distinguish the group differences"
- [readme] Evaluation pipeline functions live in GSimp_evaluation.R: "GSimp_evaluation.R contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
- [discussion] Comparison scope of the labeled evaluation: "We evaluated GSimp with other three imputation methods (kNN-TN, QRILC, and HM)"
- [abstract] MNAR context motivating supervised evaluation: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics"
