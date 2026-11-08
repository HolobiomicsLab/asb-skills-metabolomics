---
name: null-peak-exclusion-filtering
description: Use when an MSI peak matrix contains ions undetected in a high percentage
  of pixels and you must select up/down-regulated ions between ROIs or clusters (e.g.,
  cluster-vs-rest or pairwise cluster comparisons) without null peaks biasing the
  tests.
when_to_use_negative:
- Input is not an MSI peak matrix with per-pixel ion intensities and spatial clustering/ROIs
  (e.g., bulk metabolomics feature tables lack the null-peak pixel structure this
  filter addresses)
- You only need to visualize demo results without reproducing the filtering — just
  load results.Rdata instead
- No null/undetected peaks are present (all ions detected in every pixel), so the
  zeroThreshold exclusion is unnecessary
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: rMSIKeyIon
  role: Provides TestIonSelect, which implements the three-parameter filtering (null-peak
    exclusion via zeroThreshold, Mann-Whitney U test, fold-change) to select significant
    ions between MSI regions
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the MSI peak matrix (LoadPeakMatrix) and visualizes clustering and selected
    ions (plotClusterImage, plotPeakImage)
  repo: https://github.com/prafols/rMSIproc
- name: kmeans
  role: Generates the clustering vector (centers = 7) that defines the ROIs compared
    by the filtering
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:973514db36f6082fcf433fcb5ca8b144e9806ff8e93b79d21e94b77b0e8ad19e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - R
  - kmeans
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# null-peak-exclusion-filtering

## Summary

Discards undetected MS signals (null peaks) from ion concentration histograms before computing Mann-Whitney U and fold-change statistics in mass spectrometry imaging (MSI) workflows, improving reliability of up/down-regulated ion selection with rMSIKeyIon.

## When to use

Use when an MSI peak matrix contains ions undetected in a high percentage of pixels and you must select up/down-regulated ions between ROIs or clusters (e.g., cluster-vs-rest or pairwise cluster comparisons) without null peaks biasing the tests.

## When NOT to use

- Input is not an MSI peak matrix with per-pixel ion intensities and spatial clustering/ROIs (e.g., bulk metabolomics feature tables lack the null-peak pixel structure this filter addresses)
- You only need to visualize demo results without reproducing the filtering — just load results.Rdata instead
- No null/undetected peaks are present (all ions detected in every pixel), so the zeroThreshold exclusion is unnecessary

## Inputs

- MSI peak matrix loaded from Brain_peak_matrix.zip via rMSIproc::LoadPeakMatrix or peak_matrix.Rdata
- Clustering vector of pixel labels (k-means centers = 7 on peak_matrix$intensity, or clustering.Rdata)
- percentile parameter c(1,10,10)
- zeroThreshold = 2.5e-4 defining undetected (null) peaks

## Outputs

- results.Rdata containing a two-list object: 'ions' (up/down-regulated ion m/z lists per cluster comparison) and 'data' (per-ion statistics for each comparison)
- Ranked ion lists, e.g., 11 up-regulated ions in C2 (top m/z 835.656, 806.633) and 34 down-regulated ions in C3 (top m/z 868.459, 853.471); 63 up / 16 down for C2-vs-C6

## How to apply

Load the LDI-MS peak matrix (e.g., Brain_peak_matrix.zip via rMSIproc::LoadPeakMatrix) and a clustering vector (k-means with centers = 7, or clustering.Rdata). Call rMSIKeyIon::TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4; the zeroThreshold marks pixels as undetected (null) below this intensity so null peaks are excluded from the histogram before the Mann-Whitney U test and ion concentration fold-change are computed. The three-parameter filter combines (a) percentage of pixels in which an ion is not detected, (b) Mann-Whitney U ion concentration test, and (c) ion concentration fold-change. The function returns a two-list results object: 'ions' (up/down-regulated ion m/z lists per comparison) and 'data' (per-ion statistics), which should be saved as results.Rdata.

## Related tools

- **rMSIKeyIon** (Provides TestIonSelect, which implements the three-parameter filtering (null-peak exclusion via zeroThreshold, Mann-Whitney U test, fold-change) to select significant ions between MSI regions) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the MSI peak matrix (LoadPeakMatrix) and visualizes clustering and selected ions (plotClusterImage, plotPeakImage)) — https://github.com/prafols/rMSIproc
- **kmeans** (Generates the clustering vector (centers = 7) that defines the ROIs compared by the filtering)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Cluster-vs-rest comparison must yield exactly 11 absolutely up-regulated ions in cluster C2 (top m/z 835.656 and 806.633) and 34 absolutely down-regulated ions in cluster C3 (top m/z 868.459 and 853.471)
- Pairwise C2-vs-C6 comparison must contain exactly 63 up-regulated and 16 down-regulated ions ordered by the contrast parameter, matching the values distributed in the rMSIKeyIon repository's results.Rdata
- The returned object must be an R list of exactly two lists named 'ions' and 'data', one entry per cluster comparison
- Cross-check resulting m/z lists against reference values stored in the LlucSF/rMSIKeyIon GitHub repository
- Verify that ions below zeroThreshold = 2.5e-4 are excluded from the histogram before the Mann-Whitney U and fold-change calculations

## Limitations

- A high percentage of pixels with undetected MS ions challenges the reliability of statistical tests, which is precisely why null peaks must be discarded before computing p-value and fold-change
- Requires the GSL library (RcppGSL) to build; installation may be non-trivial, especially on Windows
- Document lacks a discussion section, so limitations and reproducibility gaps beyond those stated cannot be extracted

## Evidence

- [abstract] null-peak exclusion rationale: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [abstract] three-parameter filter: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test, and (c) the ion concentration"
- [methods] TestIonSelect invocation: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,"
- [readme] results structure: "The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data belonging to each ion at each"
- [abstract] null-peak difficulty: "Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels."
