---
name: null-peak-concentration-thresholding
description: Use when analyzing LDI/MSI peak matrices where many ions are undetected
  in a high percentage of pixels, which undermines the reliability of statistical
  tests comparing two ROIs.
when_to_use_negative:
- Input is already a filtered feature table with no undetected (null) pixels — the
  null-segregation step adds no value and parameter (a) is uninformative
- The goal is general clustering or ion-image visualization rather than statistical
  comparison of ions between two ROIs (use rMSIproc::plotClusterImage/plotPeakImage
  instead)
- Data are not in an rMSIproc peak_matrix structure and converting them would require
  reprocessing from raw LDI-MS images
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: rMSIKeyIon
  role: Implements TestIonSelect, which performs the null-segregation and three-parameter
    (a/b/c) ion filtering between ROIs
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix (LoadPeakMatrix) that TestIonSelect consumes
  repo: https://github.com/prafols/rMSIproc
- name: RcppGSL
  role: Links the package's C++ code to R; the package requires the GSL library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:2682e5efdaeb8948b229092769acc57f9e986a66aa32c1afe41aeb611e203931
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# null-peak-concentration-thresholding

## Summary

Handles undetected MS ions (null peaks) in metabolomic MSI images by thresholding concentrations at a zero-concentration cutoff (zeroThreshold = 2.5e-4 in the rMSIKeyIon demo), segregating null from detected pixels per ion before statistical testing.

## When to use

Use when analyzing LDI/MSI peak matrices where many ions are undetected in a high percentage of pixels, which undermines the reliability of statistical tests comparing two ROIs.

## When NOT to use

- Input is already a filtered feature table with no undetected (null) pixels — the null-segregation step adds no value and parameter (a) is uninformative
- The goal is general clustering or ion-image visualization rather than statistical comparison of ions between two ROIs (use rMSIproc::plotClusterImage/plotPeakImage instead)
- Data are not in an rMSIproc peak_matrix structure and converting them would require reprocessing from raw LDI-MS images

## Inputs

- rMSIproc peak matrix (peak_matrix$intensity structure, e.g. loaded from Brain_peak_matrix.zip or peak_matrix.Rdata)
- Two-ROI cluster/segmentation vector (e.g. from kmeans(peak_matrix$intensity, centers = 7)$cluster or clustering.Rdata)
- zeroThreshold value (zero concentration threshold, e.g. 2.5e-4)
- percentile thresholds vector (e.g. c(1,10,10))

## Outputs

- results.Rdata containing an R list of two sub-lists: 'ions' (up/down-regulated ions per ROI comparison) and 'data' (per-ion data per comparison)

## How to apply

For each ion in the rMSIproc peak_matrix$intensity structure, compute parameter (a): the percentage of pixels in which the ion is not detected, using a zero-concentration threshold (zeroThreshold = 2.5e-4 in the demo call). Then segregate null pixels from detected pixels per ion, discarding undetected signals from the histogram before computing parameter (b), the Mann-Whitney U ion concentration test between the two ROIs, and parameter (c), the fold-change of null-segregated median concentrations. The rationale is that retaining nulls distorts the Mann-Whitney U test and fold-change estimates; the null proportion is kept as a separate parameter. Finally, apply percentile-based discriminant thresholds (percentile = c(1,10,10) in the demo) to combine the three parameters and label ions as up- or down-regulated between ROIs.

## Related tools

- **rMSIKeyIon** (Implements TestIonSelect, which performs the null-segregation and three-parameter (a/b/c) ion filtering between ROIs) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix (LoadPeakMatrix) that TestIonSelect consumes) — https://github.com/prafols/rMSIproc
- **RcppGSL** (Links the package's C++ code to R; the package requires the GSL library)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- TestIonSelect executes end-to-end on a synthetic peak matrix + two-ROI cluster vector self-test and returns a correctly structured two-sub-list result ('ions' and 'data')
- Results written to results.Rdata match the demo output structure (up/down-regulated ion lists per comparison plus per-ion data)
- Null pixels identified by zeroThreshold = 2.5e-4 are excluded from the Mann-Whitney U test and fold-change histograms, while null proportions are retained as parameter (a)
- Ion labels (up/down-regulated) are consistent with the percentile thresholds c(1,10,10) applied to the three parameters

## Limitations

- The elevated number of undetected MS ions in a high percentage of pixels is an acknowledged difficulty for statistical test reliability
- No discussion/changelog text was available, so reproducibility gaps and limitations beyond those stated in the abstract cannot be extracted
- Requires the GSL library (binary download needed on Windows) and specific pinned GitHub refs of rMSI (0.8) and rMSIproc (0.2) for installation

## Evidence

- [abstract] by using a combination of three parameters: "Our ion filtering algorithm aims to find the up or down-regulated ions between two ROIs by using a combination of three parameters"
- [abstract] null peaks discarded before b and c: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [abstract] many undetected ions: "Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels."
- [readme] TestIonSelect demo call: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] results structure: "The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data belonging to each ion"
