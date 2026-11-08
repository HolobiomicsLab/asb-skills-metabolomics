---
name: fold-change-median-calculation
description: Use when filtering up/down-regulated ions between two MSI ROIs and you
  need the direction/magnitude of change (parameter c) computed on null-discarded
  median concentrations, combined with the null-ratio (a) and Mann-Whitney U (b) parameters.
when_to_use_negative:
- When you only need a binary significant/non-significant call and can ignore undetected
  peaks — the paper's rationale for the median fold-change on null-segregated data
  is the elevated number of undetected MS ions in a high percentage of pixels; classical
  p-value/FC alone misses that difficulty.
- When the input is already a summary feature table rather than a pixel-by-ion rMSIproc
  peak matrix — the fold-change is defined per ion over pixel-level null-segregated
  concentrations of two ROIs.
- When comparing more than two ROIs in a single test — the workflow defines fold-change
  between a pair of regions at a time.
edam_topics:
- http://edamontology.org/topic_3389
- http://edamontology.org/topic_2269
tools:
- name: rMSIKeyIon
  role: Package whose TestIonSelect function computes the null-ratio, Mann-Whitney
    U and median fold-change parameters and combines them into up/down-regulated ion
    labels.
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix (LoadPeakMatrix) whose intensity matrix supplies per-ion
    pixel concentrations for the fold-change computation.
  repo: https://github.com/prafols/rMSIproc
- name: Rcpp
  role: Links the package's C++ implementation of the filtering algorithm to R.
- name: RcppGSL
  role: Links R to the GSL library required by the package's C++ code.
- name: GSL
  role: Underlying numerical library required to build and run rMSIKeyIon.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:eb8c04ad55b5801c8838014836c7fe52ddf3f1a9f80628779c5fbd3902123f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - GSL
  - R
  - Rcpp
  - RcppGSL
  - rMSIKeyIon
  - rMSIproc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3389
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# fold-change-median-calculation

## Summary

Computes parameter (c) of the rMSIKeyIon three-parameter ion filter: the fold-change of medians of null-segregated ion concentrations between two regions of interest (ROIs) in an MSI peak matrix, after undetected (null) peaks are discarded.

## When to use

Use when filtering up/down-regulated ions between two MSI ROIs and you need the direction/magnitude of change (parameter c) computed on null-discarded median concentrations, combined with the null-ratio (a) and Mann-Whitney U (b) parameters.

## When NOT to use

- When you only need a binary significant/non-significant call and can ignore undetected peaks — the paper's rationale for the median fold-change on null-segregated data is the elevated number of undetected MS ions in a high percentage of pixels; classical p-value/FC alone misses that difficulty.
- When the input is already a summary feature table rather than a pixel-by-ion rMSIproc peak matrix — the fold-change is defined per ion over pixel-level null-segregated concentrations of two ROIs.
- When comparing more than two ROIs in a single test — the workflow defines fold-change between a pair of regions at a time.

## Inputs

- rMSIproc peak_matrix object (e.g. loaded via rMSIproc::LoadPeakMatrix from Brain_peak_matrix.zip or peak_matrix.Rdata) with peak_matrix$intensity pixel-by-ion matrix
- ROI/cluster vector assigning each pixel to a region (e.g. from kmeans(peak_matrix$intensity, centers = 7)$cluster or clustering.Rdata)
- zeroThreshold scalar (demo 2.5e-4) separating undetected (null) pixels from detected pixels
- percentile vector c(1,10,10) setting discriminant thresholds for parameters (a), (b), (c)

## Outputs

- R list with sub-list 'ions': up/down-regulated ions found at each two-ROI comparison
- R list with sub-list 'data': per-ion data (including null-segregated medians, fold-change and test statistics) at each comparison
- results.Rdata file persisting the two sub-lists

## How to apply

For each ion in the rMSIproc peak_matrix$intensity structure, first split pixels into null (below zeroThreshold = 2.5e-4 in the demo) and detected groups, and compute parameter (a) as the percentage of pixels in which the ion is not detected. Discard the null pixels per ion before further computation, mirroring the paper's 'third novel method to treat undetected MS peaks more accurately'. Then compute parameter (b), the Mann-Whitney U test on the null-segregated concentrations of the two ROIs, and parameter (c), the fold-change of the medians of those null-segregated concentrations between the two ROIs. Combine the three parameters via percentile-based discriminant thresholds (demo: percentile = c(1,10,10)) to label each ion as up- or down-regulated. Apply this per ROI comparison via rMSIKeyIon::TestIonSelect, which returns an R list with sub-lists 'ions' (classified ions per comparison) and 'data' (per-ion data per comparison) saved to results.Rdata.

## Related tools

- **rMSIKeyIon** (Package whose TestIonSelect function computes the null-ratio, Mann-Whitney U and median fold-change parameters and combines them into up/down-regulated ion labels.) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix (LoadPeakMatrix) whose intensity matrix supplies per-ion pixel concentrations for the fold-change computation.) — https://github.com/prafols/rMSIproc
- **Rcpp** (Links the package's C++ implementation of the filtering algorithm to R.)
- **RcppGSL** (Links R to the GSL library required by the package's C++ code.)
- **GSL** (Underlying numerical library required to build and run rMSIKeyIon.)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Null pixels below zeroThreshold (2.5e-4) are excluded per ion before medians and fold-change are computed — verify medians are taken over detected pixels only.
- Fold-change is computed from the medians (not means) of null-segregated concentrations of the two ROIs.
- Output object is an R list of exactly two sub-lists, 'ions' and 'data', keyed per two-ROI comparison.
- Ion labels (up/down-regulated) match the percentile thresholds percentile = c(1,10,10) applied to the three parameters.
- A synthetic peak matrix plus two-ROI cluster vector self-test executes end-to-end and returns correctly structured classifications.

## Limitations

- The elevated number of undetected MS ions in a high percentage of pixels undermines reliability of plain statistical tests, motivating but also constraining the null-segregation step.
- Zeroing behavior depends on the chosen zeroThreshold (demo 2.5e-4); a different threshold changes which pixels are discarded before the median fold-change.
- No discussion/changelog text was present in the source document, so limitations beyond the abstract's stated difficulty cannot be extracted.

## Evidence

- [abstract] Undetected MS signals discarded before (b)/(c): "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [abstract] Three-parameter combination: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test"
- [intro] Classical p-value/FC plus novel null-handling method: "It uses the classical p-value/ FC method and introduces a third novel method to treat undetected MS peaks more accurately."
- [readme] README demo invocation with thresholds: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,"
- [readme] Output list structure: "The first one, called "ions" contains the up/down-regulated ions"
