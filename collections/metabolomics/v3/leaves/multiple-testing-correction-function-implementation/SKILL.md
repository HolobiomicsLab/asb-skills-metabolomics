---
name: multiple-testing-correction-function-implementation
description: 'Use when you need Meff estimates for eigen-analysis-based multiple testing
  correction: you have M correlated tests (e.g., metabolites) and a Pearson correlation
  matrix, and must compute effective numbers of tests for p-value adjustment as in
  DisCo P-ad.'
when_to_use_negative:
- You only need the Meff value itself and no implementation — use the paper's reported
  results or the DisCoPad repository scripts instead of re-coding.
- Your multiple testing correction does not rely on eigen-analysis of a correlation
  matrix (e.g., simple FDR without Meff) — the seven estimators are specific to this
  family of procedures.
- The number of tests M is trivially small or features are uncorrelated — setting
  Meff = M (Bonferroni/Šidák) suffices and eigen-analysis adds no value.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implement and run the Meff estimator functions (eigen()/prcomp() for PCA of
    the correlation matrix); example scripts provided in the Codes-Explained folder
  repo: github.com/KechrisLab/DisCoPad
- name: MATLAB
  role: Alternative language for the same Meff estimator functions using eig() for
    the eigen-analysis; example scripts for one data sub-sample scenario provided
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:fa3d4f5edebfc9e2b33f19ff90633315eb85266cbbeeb3224c28eb84bcb90d29
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# multiple-testing-correction-function-implementation

## Summary

Implement the seven effective-number-of-tests (Meff) estimators used for multiple testing correction in metabolomics/GWAS contexts — Bonferroni, Šidák, Nyholt, Li and Ji, Gao, Galwey, and Peluso — as callable R or MATLAB functions built on eigen-analysis of the features' M-by-M Pearson correlation matrix (Equations 1–4e of the DisCo P-ad paper).

## When to use

Use when you need Meff estimates for eigen-analysis-based multiple testing correction: you have M correlated tests (e.g., metabolites) and a Pearson correlation matrix, and must compute effective numbers of tests for p-value adjustment as in DisCo P-ad.

## When NOT to use

- You only need the Meff value itself and no implementation — use the paper's reported results or the DisCoPad repository scripts instead of re-coding.
- Your multiple testing correction does not rely on eigen-analysis of a correlation matrix (e.g., simple FDR without Meff) — the seven estimators are specific to this family of procedures.
- The number of tests M is trivially small or features are uncorrelated — setting Meff = M (Bonferroni/Šidák) suffices and eigen-analysis adds no value.

## Inputs

- Number of tests M (scalar), for the Bonferroni and Šidák estimators
- M-by-M Pearson correlation matrix of the features/tests (for eigen-analysis-based estimators)
- Vector of eigenvalues from PCA/eigen-decomposition of that correlation matrix
- Synthetic validation matrices, e.g., identity and equicorrelation matrices

## Outputs

- Seven scalar Meff estimates, one per estimator (Bonferroni, Šidák, Nyholt, Li and Ji, Gao, Galwey, Peluso)
- Callable functions with a consistent signature (correlation matrix or M in, scalar Meff out) in R and/or MATLAB

## How to apply

Read Equations 1–3 and the multi-part Equation 4 (sub-equations 4a–4e) in Section 2.1 to extract the exact algebraic form of each of the seven Meff estimators. Bonferroni and Šidák simply set Meff equal to M (the number of tests), while the other five (Nyholt, Li and Ji, Gao, Galwey, Peluso) derive Meff from eigen-analysis of the features' M-by-M Pearson correlation matrix via PCA — e.g., eigen() or prcomp() in R, eig() in MATLAB — using the eigenvalues in Equations 4a–4e. Implement each estimator as a callable function taking the correlation matrix (or M) and returning a scalar Meff estimate, with the Equation 4 variant chaining sub-equations 4a–4e internally under one signature. Validate each function on synthetic inputs such as the identity and equicorrelation matrices, confirming estimates stay bounded within [1, M]. Reference implementations in both R and MATLAB for one simulation sub-sample scenario are available in the Codes-Explained folder of the KechrisLab/DisCoPad repository.

## Related tools

- **R** (Implement and run the Meff estimator functions (eigen()/prcomp() for PCA of the correlation matrix); example scripts provided in the Codes-Explained folder) — github.com/KechrisLab/DisCoPad
- **MATLAB** (Alternative language for the same Meff estimator functions using eig() for the eigen-analysis; example scripts for one data sub-sample scenario provided) — github.com/KechrisLab/DisCoPad

## Examples

```
lambda <- eigen(cor(X), symmetric = TRUE, only.values = TRUE)$values  # PCA eigen-analysis feeding the Meff estimators (Eqs. 1-4e)
```

## Evaluation signals

- Each function runs without error on a small synthetic correlation matrix (e.g., identity and equicorrelation matrices).
- Every estimator returns a scalar Meff bounded within [1, M] on the test inputs, consistent with theoretical range.
- Bonferroni and Šidák outputs equal M exactly, matching their closed-form definition.
- Eigen-analysis step (eigen()/prcomp() in R, eig() in MATLAB) reproduces the eigenvalues of the M-by-M Pearson correlation matrix used in Equations 4a–4e.
- Results for one simulation sub-sample scenario can be compared against the example R and MATLAB codes in the DisCoPad 'Codes-Explained' folder.

## Limitations

- Example R and MATLAB codes in the repository cover only one data sub-sample scenario used for the simulation, so edge-case behavior must be validated independently.
- Five of the seven estimators require eigen-analysis of the M-by-M correlation matrix, which can be computationally demanding when M (number of metabolite features/tests) is large.
- Estimator applicability assumes the tests' dependence structure is captured by the Pearson correlation matrix; the paper's simulations vary sample size-to-feature ratios, response types, and metabolite groupings, which can change which correction performs best.

## Evidence

- [full_text] The seven Meff estimators split into two types: Bonferroni and Šidák simply set Meff equal to M (the number of tests), while the other five derive Meff from eigen-analysis of the features' M-by-M correlation matrix: [evidence span withheld — anchor status: not_found]
- [full_text] decompose the M-by-M Pearson correlation matrix via PCA (e.g., eigen()/prcomp() in R or eig() in MATLAB) to obtain the eigenvalues used by all estimators: [evidence span withheld — anchor status: not_found]
- [full_text] confirm each function executes without error and returns an M-eff estimate bounded within [1, M] for test inputs such as the identity and equicorrelation matrices: [evidence span withheld — anchor status: not_found]
- [intro] Repository that contains links to all the codes and data-files used in the article.: "Repository that contains links to all the codes and data-files used in the article."
- [readme] Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation.: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
- [abstract] Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation.: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
