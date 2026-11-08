---
name: r-package-function-implementation
description: Use when reconstructing or re-implementing an R package function whose
  statistical mechanism must be validated against a synthetic fixture — here, a three-parameter
  MSI ion filter with null-signal segregation.
when_to_use_negative:
- Input is not a pixel-by-peak intensity matrix from an MSI dataset (e.g. already-summarized
  feature tables with no per-pixel null information)
- The analysis goal requires visualizing ions or cluster images rather than filtering
  them — rMSIproc plotting functions cover that
- Only the demo results are needed; loading the pre-computed results.Rdata avoids
  re-implementing the function
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIKeyIon
  role: Package whose TestIonSelect function implements the three-parameter filtering
    algorithm being reconstructed and tested
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Provides the peak_matrix$intensity input structure and visualization of clusters/ions
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: MSI data handling environment used alongside rMSIproc for loading and imaging
    peak matrices
  repo: https://github.com/prafols/rMSI
- name: Rcpp
  role: Links the C++ code of rMSIKeyIon to R
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: RcppGSL
  role: Binds the GSL library for the C++ statistics (Mann–Whitney U) used by rMSIKeyIon
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: GSL
  role: C statistical library required to compile the package's C++ code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:46cba02c515bc23a1dfd519cb3e79f941ab1c9f3d71a55ca1dbd7a2a64446789
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - GSL
  - R
  - Rcpp
  - RcppGSL
  - rMSI
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# r-package-function-implementation

## Summary

Implements and self-tests an R package function (TestIonSelect) that reproduces the rMSIKeyIon three-parameter ion-filtering algorithm, combining null-pixel proportion, null-segregated Mann–Whitney U test, and median fold-change to flag up/down-regulated ions between two ROIs in an rMSIproc peak matrix.

## When to use

Use when reconstructing or re-implementing an R package function whose statistical mechanism must be validated against a synthetic fixture — here, a three-parameter MSI ion filter with null-signal segregation.

## When NOT to use

- Input is not a pixel-by-peak intensity matrix from an MSI dataset (e.g. already-summarized feature tables with no per-pixel null information)
- The analysis goal requires visualizing ions or cluster images rather than filtering them — rMSIproc plotting functions cover that
- Only the demo results are needed; loading the pre-computed results.Rdata avoids re-implementing the function

## Inputs

- rMSIproc peak matrix (peak_matrix$intensity) or an equivalent synthetic R matrix emulating it
- Clustering vector or ROI assignment over pixels (e.g. from kmeans with centers = 7)
- zeroThreshold numeric cutoff for undetected signal (demo: 2.5e-4)
- percentile vector of discriminant thresholds (demo: c(1,10,10))
- Optional clusterSubset restricting comparisons to specific ROI pairs

## Outputs

- R list with sub-list 'ions': up/down-regulated ions per ROI comparison
- R list with sub-list 'data': per-ion statistics per comparison
- results.Rdata file saved from the run

## How to apply

Generate a synthetic peak matrix with an rMSIproc peak_matrix$intensity structure plus a two-ROI cluster vector as a self-test fixture. Implement TestIonSelect(PeakMtx, clusters, zeroThreshold, percentile, clusterSubset) computing per ion: (a) the percentage of pixels with undetected signal using zeroThreshold = 2.5e-4 (demo setting); (b) the Mann–Whitney U test on ion concentrations between the two ROIs after discarding null peaks from the histogram; and (c) the fold-change of medians of the null-segregated concentrations. Combine the three parameters with percentile-based discriminant thresholds (demo: percentile = c(1,10,10)) to label ions as up- or down-regulated. The rationale for discarding undetected signals before (b) and (c) is that a high percentage of undetected MS ions undermines the reliability of classical statistical tests. Return an R list of two sub-lists ('ions' with up/down-regulated ions per comparison, 'data' with per-ion data per comparison), save to results.Rdata, and run the synthetic self-test end-to-end to confirm correct structure.

## Related tools

- **rMSIKeyIon** (Package whose TestIonSelect function implements the three-parameter filtering algorithm being reconstructed and tested) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Provides the peak_matrix$intensity input structure and visualization of clusters/ions) — https://github.com/prafols/rMSIproc
- **rMSI** (MSI data handling environment used alongside rMSIproc for loading and imaging peak matrices) — https://github.com/prafols/rMSI
- **Rcpp** (Links the C++ code of rMSIKeyIon to R) — https://github.com/LlucSF/rMSIKeyIon
- **RcppGSL** (Binds the GSL library for the C++ statistics (Mann–Whitney U) used by rMSIKeyIon) — https://github.com/LlucSF/rMSIKeyIon
- **GSL** (C statistical library required to compile the package's C++ code)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- The function returns a list of exactly two sub-lists, 'ions' and 'data', with per-comparison entries
- The synthetic self-test runs end-to-end without errors and yields correctly structured up/down classifications
- Null peaks are verifiably excluded from the Mann–Whitney U test and fold-change computations (parameter segregation check)
- Demo parameters zeroThreshold = 2.5e-4 and percentile = c(1,10,10) are honored as function defaults/arguments
- Output matches the structure of the demo results.Rdata when run on the bundled Brain_peak_matrix fixture

## Limitations

- A high percentage of undetected MS ions across pixels undermines classical statistical-test reliability, which is why null segregation is mandatory before parameters (b) and (c)
- Requires the GSL library (binary download on Windows) and devtools/GitHub installation of pinned refs rMSI 0.8 and rMSIproc 0.2
- No changelog/limitations discussion is present in the source article, so edge-case behavior beyond the demo settings is undocumented

## Evidence

- [abstract] Three-parameter filtering: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test, and (c) the ion concentration"
- [abstract] Null-signal segregation rationale: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [abstract] Reliability caveat: "Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels."
- [readme] Demo invocation: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] Output structure: "The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data belonging to each ion at each comparison."
- [readme] Input structure: "new_clustering_vector <- kmeans(peak_matrix$intensity, centers = 7)$cluster"
