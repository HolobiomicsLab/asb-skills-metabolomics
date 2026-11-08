---
name: nearest-spd-matrix-projection
description: Use when an empirical M-by-M pairwise distance-correlation matrix of
  metabolite features must be made non-negative-definite before eigen-analysis-based
  effective-number-of-tests (Meff) correction in a MWAS multiple-testing pipeline.
when_to_use_negative:
- The input matrix is already verified non-negative definite (e.g., a Pearson correlation
  matrix known to be PSD) — projection would be redundant
- The downstream correction does not rely on eigen-analysis of the correlation matrix,
  so PSD-ness is not required
- The input is a raw samples-by-metabolites feature table rather than a square pairwise
  distance-correlation matrix
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MATLAB
  role: Hosts the fastDcov routine for computing the DisCo matrix and the nearestSPD
    routine used for the Frobenius-norm projection to non-negative definiteness
  repo: github.com/KechrisLab/DisCoPad
- name: R
  role: Used alongside MATLAB in the DisCo P-ad example scripts from the Codes-Explained
    folder of the repository
  repo: github.com/KechrisLab/DisCoPad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010028
    title: 'DisCo P-ad: Distance-Correlation-Based p-Value Adjustment Enhances Multiple
      Testing Corrections for Metabolomics'
  source_content_hash: sha256:87818e6328da555a6c6afcb4c00438da7742fbb1834a56d395bcf66ec6adda30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nearest-spd-matrix-projection

## Summary

Projects an empirical distance-correlation (DisCo) matrix onto the nearest non-negative-definite matrix in the Frobenius norm using the MATLAB nearestSPD() routine, so that eigen-analysis-based Meff estimation in the DisCo P-ad pipeline is mathematically valid.

## When to use

Use when an empirical M-by-M pairwise distance-correlation matrix of metabolite features must be made non-negative-definite before eigen-analysis-based effective-number-of-tests (Meff) correction in a MWAS multiple-testing pipeline.

## When NOT to use

- The input matrix is already verified non-negative definite (e.g., a Pearson correlation matrix known to be PSD) — projection would be redundant
- The downstream correction does not rely on eigen-analysis of the correlation matrix, so PSD-ness is not required
- The input is a raw samples-by-metabolites feature table rather than a square pairwise distance-correlation matrix

## Inputs

- Empirical M-by-M pairwise distance-correlation (DisCo) matrix of metabolite features, computed with the fastDcov MATLAB routine
- Samples-by-metabolites metabolomics feature matrix produced upstream

## Outputs

- Non-negative-definite M-by-M DisCo matrix written to disco_pad_matrix.csv
- Meff values computed from the projected matrix via eigen-analysis-based estimators

## How to apply

First compute the M-by-M empirical DisCo matrix using the fastDcov routine in MATLAB on the samples-by-metabolites feature matrix from upstream (task_001 context). Then apply the nearestSPD() function, which finds the nearest non-negative definite matrix with respect to the Frobenius norm: eigenvalues below the 1e-12 threshold are zeroed before reconstruction. The rationale is that the empirical DisCo matrix is not guaranteed to be non-negative definite, but the Meff estimators rely on eigen-analysis, which requires a valid (non-negative-definite) correlation-like matrix. Pass the projected matrix to the eigen-analysis-based Meff estimators (originally defined for Pearson correlation matrices) so the multiple-testing correction runs on distance correlations instead. Write the adjusted matrix to disco_pad_matrix.csv for downstream validation and reuse.

## Related tools

- **MATLAB** (Hosts the fastDcov routine for computing the DisCo matrix and the nearestSPD routine used for the Frobenius-norm projection to non-negative definiteness) — github.com/KechrisLab/DisCoPad
- **R** (Used alongside MATLAB in the DisCo P-ad example scripts from the Codes-Explained folder of the repository) — github.com/KechrisLab/DisCoPad

## Examples

```
B = nearestSPD(A); % project empirical DisCo matrix A onto nearest non-negative-definite matrix
```

## Evaluation signals

- disco_pad_matrix.csv exists and is a symmetric M-by-M matrix
- The projected matrix is non-negative-definite: minimum eigenvalue >= 0 after thresholding at 1e-12
- Meff values computed from the projected matrix are consistent with the eigen-analysis-based estimators from task_001
- The projection is the nearest matrix in the Frobenius norm (minimal Frobenius distance to the original empirical DisCo matrix)

## Limitations

- The projection zeroes eigenvalues below the 1e-12 threshold, discarding near-zero eigen-structure that may carry weak information
- Validation is per sub-sample scenario; the repository only ships example R and MATLAB codes for one data sub-sample scenario used in the simulation

## Evidence

- [methods] Nearest SPD projection: [evidence span withheld — anchor status: not_found]
- [methods] 1e-12 eigenvalue threshold: [evidence span withheld — anchor status: not_found]
- [abstract] Purpose: eigen-analysis Meff: "We propose the distance-correlation-based p-value adjustment (DisCo P-ad) as a novel modification that can enhance existing eigen-analysis-based multiple testing correction procedures by increasing"
- [readme] Repository scripts location: "Check out "Codes-Explained" folder to find all the example R and MATLAB codes (for one data sub-sample scenario) used for the simulation."
- [results] Validation of output matrix: [evidence span withheld — anchor status: not_found]
