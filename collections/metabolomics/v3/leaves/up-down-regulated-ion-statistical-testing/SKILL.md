---
name: up-down-regulated-ion-statistical-testing
description: Use when an MSI peak matrix has been segmented into clusters/ROIs and
  you need the statistically significant up- or down-regulated ions between two specific
  regions, with undetected (null) peaks handled explicitly.
when_to_use_negative:
- The data are already an annotated feature table with statistical test results —
  the filtering workflow expects a raw MSI peak matrix and clustering.
- The analysis is a simple p-value/fold-change test where undetected peaks are not
  an issue — rMSIKeyIon's third null-peak method targets datasets with a high percentage
  of pixels where an ion is not detected.
- The goal is image visualization only — rMSIproc::plotClusterImage/plotPeakImage
  handle that without the statistical selection step.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIKeyIon
  role: 'Core package: TestIonSelect applies the three-parameter filtering (null-peak
    percentage, Mann-Whitney U test, fold-change) to find up/down-regulated ions between
    ROIs'
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix from Brain_peak_matrix.zip and visualizes clustering
    and ion images
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Part of the rMSI environment used to visualize ions and clusters alongside
    rMSIproc
  repo: https://github.com/prafols/rMSI
- name: kmeans
  role: Unsupervised segmentation of the image (centers = 7) to produce the clustering
    vector of ROIs fed to TestIonSelect
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:ec2b56b8b69323886ca2adecb65dd4b769464c8417f57eef24bf3fac677174f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - R
  - kmeans
  - rMSI
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# up-down-regulated-ion-statistical-testing

## Summary

Statistical ion filtering for untargeted LDI-MS imaging using the rMSIKeyIon R package: it combines a null-peak threshold, a Mann-Whitney U ion concentration test, and ion concentration fold-change to report up- and down-regulated ions between pairs of image regions.

## When to use

Use when an MSI peak matrix has been segmented into clusters/ROIs and you need the statistically significant up- or down-regulated ions between two specific regions, with undetected (null) peaks handled explicitly.

## When NOT to use

- The data are already an annotated feature table with statistical test results — the filtering workflow expects a raw MSI peak matrix and clustering.
- The analysis is a simple p-value/fold-change test where undetected peaks are not an issue — rMSIKeyIon's third null-peak method targets datasets with a high percentage of pixels where an ion is not detected.
- The goal is image visualization only — rMSIproc::plotClusterImage/plotPeakImage handle that without the statistical selection step.

## Inputs

- MSI peak matrix (e.g. Brain_peak_matrix.zip loaded via rMSIproc::LoadPeakMatrix, or peak_matrix.Rdata)
- Clustering vector or ROI assignment over image pixels (e.g. from kmeans with centers = 7, or clustering.Rdata)

## Outputs

- results list with an 'ions' entry per cluster comparison containing ranked up-regulated and down-regulated ion lists (e.g. 63 up / 16 down for C2 vs C6)
- Parallel 'data' list with per-ion statistics for each comparison
- Saved results.Rdata artifact

## How to apply

Load the MSI peak matrix (e.g. rMSIproc::LoadPeakMatrix on Brain_peak_matrix.zip), then segment the image, e.g. kmeans(peak_matrix$intensity, centers = 7)$cluster, or load a provided clustering vector. Run rMSIKeyIon::TestIonSelect with the peak matrix, the clustering vector, percentile = c(1,10,10), and zeroThreshold = 2.5e-4; the zeroThreshold discards null peaks (ions undetected in a high percentage of pixels) before the Mann-Whitney U test and fold-change are computed, improving test reliability. Extract the 'ions' list entry for the desired cluster-vs-cluster comparison; each comparison yields ranked up-regulated and down-regulated ion lists ordered by the contrast parameter, with per-ion details in the parallel 'data' list. Save the two-list results object as results.Rdata and optionally cross-check m/z values against the reference results.Rdata in the LlucSF/rMSIKeyIon repository.

## Related tools

- **rMSIKeyIon** (Core package: TestIonSelect applies the three-parameter filtering (null-peak percentage, Mann-Whitney U test, fold-change) to find up/down-regulated ions between ROIs) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix from Brain_peak_matrix.zip and visualizes clustering and ion images) — https://github.com/prafols/rMSIproc
- **rMSI** (Part of the rMSI environment used to visualize ions and clusters alongside rMSIproc) — https://github.com/prafols/rMSI
- **kmeans** (Unsupervised segmentation of the image (centers = 7) to produce the clustering vector of ROIs fed to TestIonSelect)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- The C2-vs-C6 'ions' output contains exactly 63 up-regulated and 16 down-regulated ions, matching the reference results.Rdata from the rMSIKeyIon repository.
- Up- and down-regulated ion lists within each comparison are ranked by the contrast parameter.
- Null peaks above the zeroThreshold = 2.5e-4 criterion are excluded before Mann-Whitney U and fold-change computation.
- The returned object is a list of two lists named 'ions' and 'data' covering every cluster-vs-cluster comparison.
- Extracted m/z lists cross-check against the reference values distributed in the LlucSF/rMSIKeyIon GitHub results.Rdata.

## Limitations

- Statistical test reliability suffers from the elevated number of undetected MS ions in a high percentage of pixels, which is why null peaks must be discarded first.
- The package requires the GSL library (plus Rcpp and RcppGSL) to link its C++ code to R, which can complicate installation, especially on Windows.
- A clustering/ROI vector must exist before running the test; the package does not itself segment the image (the demo uses k-means with 7 centers).

## Evidence

- [abstract] Our ion filtering algorithm aims to find the up or down-regulated ions between two ROIs by using a combination of three parameters: "Our ion filtering algorithm aims to find the up or down-regulated ions between two ROIs by using a combination of three parameters"
- [abstract] The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters.: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [methods] results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4): "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data belonging to each ion at each comparison.: "The first one, called "ions" contains the up/down-regulated ions
found at each comparison. The second one, called "data" contains all the data belonging to each ion at each
comparison."
- [abstract] With this methodology, we found the important ions between the different segments of a mouse brain tissue sagittal section: "With this methodology, we found the important ions between the different segments of a mouse brain tissue sagittal section"
