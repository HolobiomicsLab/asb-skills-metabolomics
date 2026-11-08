---
name: effective-number-of-tests-estimation
description: Use when you have M metabolite features with a features-by-samples matrix
  (e.g. 761 metabolites x 1054 COPDGene subjects) and need to correct marginal association
  p-values via an effective-number-of-tests (Meff) estimate instead of raw M.
when_to_use_negative:
- Input is already a set of p-values with no feature matrix — eigen-analysis-based
  Meff estimators require the features' correlation structure, not just p-values
- Only a handful of features/tests — Bonferroni (Meff = M) suffices and eigen-analysis
  corrections add no benefit
- The correlation matrix is used for direct coefficient interpretation rather than
  p-value adjustment; nearestSPD projection alters eigenvalues and is intended only
  to enable valid eigen-analysis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Implement the seven Meff estimators and eigen-analysis via eigen()/prcomp();
    assemble RMSE comparison tables
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Compute pairwise distance correlations with fastDcov, project the DisCo matrix
    with nearestSPD (1e-12 eigenvalue threshold), and run eig()-based Meff estimation
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  - task_001
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:fd8c5a83a46f0e7cb6b840950d0fd259b8079073b75d660e2aeafe7de178bf9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# effective-number-of-tests-estimation

## Summary

Estimate the effective number of tests (Meff) in high-dimensional metabolomics multiple testing using seven estimators — Bonferroni, Šidák, Nyholt, Li and Ji, Gao, Galwey, Peluso — of which five derive Meff from eigen-analysis of an M-by-M correlation matrix, optionally recomputed on distance correlations as in DisCo P-ad.

## When to use

Use when you have M metabolite features with a features-by-samples matrix (e.g. 761 metabolites x 1054 COPDGene subjects) and need to correct marginal association p-values via an effective-number-of-tests (Meff) estimate instead of raw M.

## When NOT to use

- Input is already a set of p-values with no feature matrix — eigen-analysis-based Meff estimators require the features' correlation structure, not just p-values
- Only a handful of features/tests — Bonferroni (Meff = M) suffices and eigen-analysis corrections add no benefit
- The correlation matrix is used for direct coefficient interpretation rather than p-value adjustment; nearestSPD projection alters eigenvalues and is intended only to enable valid eigen-analysis

## Inputs

- samples x metabolites feature abundance matrix (e.g. COPDGene ST001443, 761 metabolites x 1054 subjects)
- M-by-M Pearson correlation matrix computed from the features
- M-by-M empirical distance-correlation (DisCo) matrix from fastDcov
- optional continuous outcome such as logit(FEV1/FVC) for permutation-based Meff0 benchmarks

## Outputs

- scalar Meff estimate per estimator, bounded in [1, M]
- nearest non-negative-definite DisCo matrix written as disco_pad_matrix.csv
- RMSE comparison table meff_rmse_results.csv with method, matrix type, grouping, and sample-size columns
- gold-standard table meff0_gold_standard.csv (Meff0 per sample size n=100/500/1000)

## How to apply

From the features (samples x metabolites) matrix, compute the M-by-M correlation matrix — Pearson's for the standard estimators, or the distance-correlation (DisCo) matrix built with fastDcov in MATLAB for the DisCo P-ad variant. Eigen-decompose the matrix (eig() in MATLAB or eigen()/prcomp() in R); Bonferroni and Šidák simply set Meff = M, while Nyholt, Li and Ji, Gao, Galwey, and Peluso apply the eigenvalue-based formulas of Equations 4a-4e. If using a DisCo matrix, project it to the nearest non-negative-definite matrix with nearestSPD() in the Frobenius norm, zeroing eigenvalues below the 1e-12 threshold before reconstruction. Wrap all seven estimators with a consistent signature (correlation matrix or M in, scalar Meff out). To judge accuracy, benchmark Meff estimates against a permutation-based gold standard Meff0 = alpha / alpha_f0 (K=10,000 label permutations) and compare via RMSE of median estimates per method x matrix x grouping x sample-size cell.

## Related tools

- **R** (Implement the seven Meff estimators and eigen-analysis via eigen()/prcomp(); assemble RMSE comparison tables) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Compute pairwise distance correlations with fastDcov, project the DisCo matrix with nearestSPD (1e-12 eigenvalue threshold), and run eig()-based Meff estimation) — github.com/KechrisLab/DisCoPad

## Examples

```
[V,D] = eig(R); meff = nyholt_meff(diag(V));  % R is the M-by-M (DisCo or Pearson) correlation matrix; Meff in [1, M]
```

## Evaluation signals

- Each estimator returns a scalar Meff bounded within [1, M] on test inputs such as identity and equicorrelation matrices
- disco_pad_matrix.csv is symmetric, M-by-M, and non-negative-definite (minimum eigenvalue >= 0 after thresholding at 1e-12)
- Meff values from the DisCo matrix are consistent with the eigen-analysis-based estimators from the standard pipeline
- meff_rmse_results.csv covers all 2 matrices x 5 methods x 3 groupings x 3 sample sizes cells with non-negative finite RMSE values
- Permutation-based Meff0 = alpha / alpha_f0 values are finite positive numbers whose ordering matches the paper's Table 1 row 1 for the COPDGene scenario

## Limitations

- Distance correlation matrices are not guaranteed non-negative-definite and require nearestSPD projection (Frobenius norm, 1e-12 eigenvalue floor) before eigen-analysis
- Eigen-analysis-based estimators' accuracy depends on sample size-to-feature ratios; at n=100 the RMSE ordering across methods differed substantially (Nyholt smallest for DisCo)
- The gold-standard Meff0 varies with sample size because larger n better captures null associations between metabolite abundances and the outcome

## Evidence

- [other] Seven Meff estimators split into two types: [evidence span withheld — anchor status: not_found]
- [other] DisCo matrix converted to nearest NND: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: DisCo P-ad proposal: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures"
- [readme] README: codes and data access: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
