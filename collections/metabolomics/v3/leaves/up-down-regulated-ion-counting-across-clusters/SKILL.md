---
name: up-down-regulated-ion-counting-across-clusters
description: Use when a k-means (or ROI) segmented MSI peak matrix must be compared
  one cluster versus the rest to report how many absolutely up- or down-regulated
  ions each region contains and which m/z values characterize them.
when_to_use_negative:
- When the data are not region-segmented MSI peak matrices — no clustering_vector
  or ROI definition exists for a one-cluster-versus-rest design
- 'When only visualizing demo output without running the algorithm: loading results.Rdata
  suffices and TestIonSelect should not be re-run'
- When the analysis targets targeted metabolite quantification rather than untargeted
  ion filtering between image regions
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rMSIKeyIon
  role: Core package whose TestIonSelect function performs the three-parameter ion
    filtering (null-peak percentage, Mann-Whitney U test, fold-change) for each cluster-vs-rest
    comparison
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the Brain_peak_matrix.zip peak matrix and visualizes clustering and
    ions (plotClusterImage, plotPeakImage)
  repo: https://github.com/prafols/rMSIproc
- name: kmeans
  role: Unsupervised segmentation of peak_matrix$intensity into 7 clusters when clustering.Rdata
    is not pre-computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:b2aac8c445a18e44b6baddd7e7e4dff5289015db0476d93af73d15d6b4fbbb09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - R
  - kmeans
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# up-down-regulated-ion-counting-across-clusters

## Summary

Applies rMSIKeyIon::TestIonSelect to a segmented mass spectrometry imaging peak matrix to count absolutely up- and down-regulated ions in each cluster-vs-rest comparison, using a three-parameter filter (null-peak percentage, Mann-Whitney U test, fold-change).

## When to use

Use when a k-means (or ROI) segmented MSI peak matrix must be compared one cluster versus the rest to report how many absolutely up- or down-regulated ions each region contains and which m/z values characterize them.

## When NOT to use

- When the data are not region-segmented MSI peak matrices — no clustering_vector or ROI definition exists for a one-cluster-versus-rest design
- When only visualizing demo output without running the algorithm: loading results.Rdata suffices and TestIonSelect should not be re-run
- When the analysis targets targeted metabolite quantification rather than untargeted ion filtering between image regions

## Inputs

- MSI peak matrix from Brain_peak_matrix.zip loaded with rMSIproc::LoadPeakMatrix, or peak_matrix.Rdata loaded via base R
- Cluster assignment vector from clustering.Rdata (clustering_vector, k-means with 7 centers), or a kmeans(peak_matrix$intensity, centers = 7)$cluster result

## Outputs

- results object (results.Rdata) containing the 'ions' list of up/down-regulated ions per one-cluster-versus-rest comparison
- the 'data' list with per-ion statistics (Mann-Whitney U p-value, fold-change, null-peak percentage) for each comparison
- absolute counts of up/down-regulated ions per cluster and top-contrast m/z values

## How to apply

Load the mouse brain peak matrix via rMSIproc::LoadPeakMatrix from Brain_peak_matrix.zip (or load peak_matrix.Rdata) and obtain the 7-center k-means segmentation from clustering.Rdata (clustering_vector). Run rMSIKeyIon::TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4 so that undetected (null) peaks are discarded before computing the Mann-Whitney U test and ion concentration fold-change for each one-cluster-versus-rest comparison. Extract the 'ions' list of up/down-regulated ions and the 'data' list of per-ion statistics; validate the absolute counts (e.g., 11 absolutely up-regulated ions in C2 with top m/z 835.656 and 806.633; 34 absolutely down-regulated ions in C3 with top m/z 868.459 and 853.471) and save the results object to results.Rdata.

## Related tools

- **rMSIKeyIon** (Core package whose TestIonSelect function performs the three-parameter ion filtering (null-peak percentage, Mann-Whitney U test, fold-change) for each cluster-vs-rest comparison) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the Brain_peak_matrix.zip peak matrix and visualizes clustering and ions (plotClusterImage, plotPeakImage)) — https://github.com/prafols/rMSIproc
- **kmeans** (Unsupervised segmentation of peak_matrix$intensity into 7 clusters when clustering.Rdata is not pre-computed)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- C2 comparison yields exactly 11 absolutely up-regulated ions, with top-contrast ions at m/z 835.656 and 806.633
- C3 comparison yields exactly 34 absolutely down-regulated ions, with top-contrast ions at m/z 868.459 and 853.471
- Results object contains exactly two lists: 'ions' (up/down-regulated ions per comparison) and 'data' (per-ion statistics per comparison)
- Null peaks are excluded before statistical testing — peaks with zero-intensity fraction above zeroThreshold = 2.5e-4 handling must not contribute to the Mann-Whitney U test or fold-change
- Saved results.Rdata reloads in a fresh R session and reproduces the same ion counts

## Limitations

- A high percentage of pixels with undetected MS ions undermines statistical test reliability, which is why undetected (null) peaks must be discarded before the Mann-Whitney U test and fold-change calculation
- The package requires the GSL library (via RcppGSL) to link C++ code to R, complicating installation on some systems
- No discussion of limitations or reproducibility gaps is available in the document (discussion section empty)

## Evidence

- [abstract] three-parameter filter: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test, and (c) the ion concentration"
- [abstract] null peaks discarded: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [readme] TestIonSelect invocation: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] results structure: "The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data belonging to each ion at each comparison."
- [readme] GSL dependency: [evidence span withheld — anchor status: not_found]
