---
name: metabolomics-high-dimensional-correlation-analysis
description: Use when a samples-by-metabolites high-dimensional feature matrix requires
  an M-by-M dependency matrix for eigen-analysis-based Meff estimation, and Pearson
  correlation is suspected of missing non-linear dependence among metabolite features.
when_to_use_negative:
- The goal is standard Pearson-correlation-based Meff correction (plain GWAS-style
  eigen-analysis), not the distance-correlation-based DisCo P-ad modification.
- The input is already a validated non-negative-definite DisCo matrix (disco_pad_matrix.csv)
  awaiting Meff estimation — proceed directly to the eigen-analysis step.
- The feature matrix is not high-dimensional omics-style data (samples by features)
  from which an M-by-M pairwise dependency matrix is required.
tools:
- name: MATLAB
  role: Computes all M-by-M pairwise distance correlations with fastDcov and projects
    the empirical DisCo matrix to the nearest non-negative-definite matrix with nearestSPD
    before reconstruction.
- name: R
  role: Hosts the eigen-analysis-based Meff estimators that consume the adjusted non-negative-definite
    DisCo matrix for p-value adjustment.
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:ee8f92d431d7600b8ad465fb7dfbacdebd05e7700ff2505ad1eaf98b801fd2d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-high-dimensional-correlation-analysis

## Summary

Computes an M-by-M empirical distance-correlation (DisCo) matrix from a metabolite feature matrix using MATLAB's fastDcov, projects it onto the nearest non-negative-definite matrix with nearestSPD, and feeds the result to eigen-analysis-based Meff estimators for DisCo P-ad multiple-testing correction.

## When to use

Use when a samples-by-metabolites high-dimensional feature matrix requires an M-by-M dependency matrix for eigen-analysis-based Meff estimation, and Pearson correlation is suspected of missing non-linear dependence among metabolite features.

## When NOT to use

- The goal is standard Pearson-correlation-based Meff correction (plain GWAS-style eigen-analysis), not the distance-correlation-based DisCo P-ad modification.
- The input is already a validated non-negative-definite DisCo matrix (disco_pad_matrix.csv) awaiting Meff estimation — proceed directly to the eigen-analysis step.
- The feature matrix is not high-dimensional omics-style data (samples by features) from which an M-by-M pairwise dependency matrix is required.

## Inputs

- Samples-by-metabolites metabolomics feature matrix (e.g. loaded from repository data files)
- Empirical M-by-M distance-correlation (DisCo) matrix computed via fastDcov
- Metabolite groupings / sample-size-to-feature ratio scenario definition

## Outputs

- disco_pad_matrix.csv — symmetric M-by-M non-negative-definite distance-correlation matrix (minimum eigenvalue >= 0 after 1e-12 thresholding)
- Meff (effective number of tests) values from eigen-analysis-based estimators computed on the adjusted DisCo matrix

## How to apply

Load the samples-by-metabolites feature matrix produced upstream and compute all M-by-M pairwise distance correlations with the fastDcov routine in MATLAB, yielding the empirical DisCo matrix. Because distance-correlation matrices need not be non-negative definite, apply the nearestSPD routine to project the empirical DisCo matrix onto the nearest non-negative-definite matrix with respect to the Frobenius norm, zeroing eigenvalues below the 1e-12 threshold before reconstruction. Pass the resulting non-negative-definite matrix to the eigen-analysis-based Meff estimators (referenced in R) so that effective-number-of-tests correction is recomputed on distance correlations instead of Pearson correlations, following the DisCo P-ad approach that enhances existing multiple-testing correction procedures. Write the adjusted matrix to disco_pad_matrix.csv and validate it before downstream use.

## Related tools

- **MATLAB** (Computes all M-by-M pairwise distance correlations with fastDcov and projects the empirical DisCo matrix to the nearest non-negative-definite matrix with nearestSPD before reconstruction.)
- **R** (Hosts the eigen-analysis-based Meff estimators that consume the adjusted non-negative-definite DisCo matrix for p-value adjustment.) — github.com/KechrisLab/DisCoPad

## Examples

```
% MATLAB: project empirical DisCo matrix to nearest PSD then validate; D = fastDcov(X); M = nearestSPD(D); assert(all(eig(M) >= 0))
```

## Evaluation signals

- disco_pad_matrix.csv exists and is a symmetric M-by-M matrix
- The output matrix is non-negative definite: minimum eigenvalue >= 0 after thresholding at 1e-12
- The projected matrix is the nearest non-negative-definite matrix to the empirical DisCo matrix in the Frobenius norm (no closer PSD matrix exists)
- Meff values computed from disco_pad_matrix.csv are consistent with the eigen-analysis-based estimators from the upstream Pearson-correlation workflow
- Recomputing fastDcov on the original feature matrix reproduces the pre-projection empirical DisCo matrix

## Limitations

- The empirical DisCo matrix is not guaranteed non-negative definite, so nearestSPD projection (zeroing eigenvalues below 1e-12) is a mandatory but lossy step that slightly perturbs the original distance correlations.
- The repository's 'Codes-Explained' example R and MATLAB codes cover only one data sub-sample scenario used for the simulation, not every configuration in the study.
- Scripts must be interpreted via the repository's Read-Me.txt, which documents what each script accomplishes; the link-to-codes-and-data-objects file is the entry point for codes and data.

## Evidence

- [abstract] We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures by increasing: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures by increasing"
- [readme] Click on the 'link-to-codes-and-data-objects' file to access all codes and related items.: "Click on the "link-to-codes-and-data-objects" file to access all codes and related items."
- [readme] Check out 'Codes-Explained' folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation.: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation"
