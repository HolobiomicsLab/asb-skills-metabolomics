---
name: distance-correlation-computation
description: Use when an eigen-analysis-based p-value adjustment method originally
  defined on Pearson correlations must be re-expressed on distance correlations for
  metabolomics features, or when Meff estimates need to account for nonlinear dependence
  structure.
when_to_use_negative:
- The goal is plain Pearson-correlation-based Meff correction (PrsCo) and no nonlinear
  dependence is expected — the distance correlation adds computation without benefit.
- Input is not a numeric samples-by-features metabolomics matrix (e.g., already an
  adjusted p-value table), since pairwise distance correlations require raw feature
  values per sample.
- The downstream estimator does not require a non-negative-definite matrix and you
  cannot apply nearestSPD projection, as eigen-analysis-based Meff methods need valid
  eigen-decompositions.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MATLAB
  role: Used to compute the empirical distance-correlation matrix with the fastDcov
    routine and to apply the nearestSPD projection before eigen-analysis.
  repo: https://github.com/KechrisLab/DisCoPad
- name: R
  role: Used for downstream Meff computation and RMSE comparison of DisCo P-ad vs
    PrsCo P-ad across methods, groupings, and sample sizes.
  repo: https://github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:411c237904da5a56fe7f34f1233280f41ce0c76b11789209f64e71ba50baed9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KechrisLab/DisCoPad
  tools:
  - MATLAB
  - R
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# distance-correlation-computation

## Summary

Compute all pairwise distance correlations (DisCo) among M metabolite features to build an M-by-M distance-correlation matrix that captures nonlinear dependence, then project it to the nearest non-negative-definite matrix (nearestSPD, eigenvalue threshold 1e-12) so eigen-analysis-based Meff estimation can be applied. This is the core operation enabling the DisCo P-ad multiple-testing correction for metabolomics.

## When to use

Use when an eigen-analysis-based p-value adjustment method originally defined on Pearson correlations must be re-expressed on distance correlations for metabolomics features, or when Meff estimates need to account for nonlinear dependence structure.

## When NOT to use

- The goal is plain Pearson-correlation-based Meff correction (PrsCo) and no nonlinear dependence is expected — the distance correlation adds computation without benefit.
- Input is not a numeric samples-by-features metabolomics matrix (e.g., already an adjusted p-value table), since pairwise distance correlations require raw feature values per sample.
- The downstream estimator does not require a non-negative-definite matrix and you cannot apply nearestSPD projection, as eigen-analysis-based Meff methods need valid eigen-decompositions.

## Inputs

- samples x metabolites feature matrix (metabolomics data table) from the repository data files
- PrsCo (Pearson) correlation matrix for comparison, produced upstream

## Outputs

- disco_pad_matrix.csv: symmetric, non-negative-definite M-by-M distance-correlation matrix
- Meff estimates per method x matrix x grouping x sample-size cell (feeding meff_rmse_results.csv)

## How to apply

Load the samples-by-metabolites feature matrix produced upstream, then compute all M-by-M pairwise distance correlations with the fastDcov routine in MATLAB to obtain the empirical DisCo matrix. Because the empirical distance-correlation matrix may not be non-negative definite, apply the nearestSPD routine to project it onto the nearest non-negative-definite matrix in the Frobenius norm, zeroing eigenvalues below the 1e-12 threshold before reconstruction. Pass the resulting non-negative-definite matrix to the eigen-analysis-based Meff estimators (e.g., Nyholt and related procedures, as used in the RMSE comparison against the permutation-based gold standard Meff0) so the distance-correlation-based adjustment (DisCo P-ad) can be recomputed. Write the adjusted matrix to disco_pad_matrix.csv. Rationale: the article found the distance correlation superior to Pearson correlation across varying sample size-to-feature ratios, response types, and metabolite groupings.

## Related tools

- **MATLAB** (Used to compute the empirical distance-correlation matrix with the fastDcov routine and to apply the nearestSPD projection before eigen-analysis.) — https://github.com/KechrisLab/DisCoPad
- **R** (Used for downstream Meff computation and RMSE comparison of DisCo P-ad vs PrsCo P-ad across methods, groupings, and sample sizes.) — https://github.com/KechrisLab/DisCoPad

## Examples

```
DisCo = fastDcov(X); DisCo_nnd = nearestSPD(DisCo); writematrix(DisCo_nnd, 'disco_pad_matrix.csv')
```

## Evaluation signals

- disco_pad_matrix.csv exists and is a symmetric M-by-M matrix.
- The adjusted matrix is non-negative-definite: minimum eigenvalue >= 0 after thresholding at 1e-12.
- Meff values computed from the DisCo matrix are consistent with the eigen-analysis-based estimators from the upstream task.
- The output covers all 2 matrices x 5 methods x 3 groupings x 3 sample sizes cells with non-negative finite RMSE values.
- At n=100 and all three metabolite groupings, the Nyholt method applied on DisCo should achieve the smallest RMSE against the gold-standard Meff0, outperforming other methods by large margins.

## Limitations

- The empirical distance-correlation matrix is not guaranteed non-negative definite, so nearestSPD projection with a 1e-12 eigenvalue threshold is mandatory before eigen-analysis.
- Performance conclusions (e.g., Nyholt with DisCo achieving smallest RMSE) were established for specific simulation settings: sample sizes including n=100 and the three metabolite groupings (ungrouped, defined 8-pathway, random); they may not transfer to all metabolomics scenarios.
- Example codes in the repository cover only one data sub-sample scenario used for the simulation.

## Evidence

- [abstract] superior performance of the distance correlation: "Our study, including varying sample size-to-feature ratios, response types, and metabolite groupings, highlights the superior performance of the distance correlation."
- [abstract] DisCo P-ad as enhancement of eigen-analysis methods: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures"
- [intro] example codes location: "find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation"
- [readme] repository links to codes and data: "Repository that contains links to all the codes and data-files used in the article."
