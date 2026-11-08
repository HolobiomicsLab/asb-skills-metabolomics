---
name: mann-whitney-u-hypothesis-testing
description: Use when comparing ion concentrations between two regions of interest
  in an MSI peak matrix where undetected (null) peaks would otherwise invalidate parametric
  testing and must be discarded before the test.
when_to_use_negative:
- Input is already a filtered feature table rather than an rMSIproc peak matrix with
  pixel-level ion intensities
- The comparison involves more than two regions at once without pairwise comparisons,
  since the test is defined between two ROIs
- Data are continuous, fully detected, and normally distributed, where a parametric
  test on the raw concentrations would be preferable
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rMSIKeyIon
  role: Implements TestIonSelect, which performs the null-segregated Mann–Whitney
    U test and combines it with the null-ratio and fold-change parameters
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix and visualizes the clustering and ions used as test
    input
  repo: https://github.com/prafols/rMSIproc
- name: RcppGSL
  role: Links the C++ code of rMSIKeyIon to R, requiring the GSL library
- name: R
  role: Runtime environment; kmeans is used to generate the clustering vector of ROIs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:abadb3449e10ed9aacb711d7b2b5b27b38b931d66cb8db384c703c831323b6f8
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mann-whitney-u-hypothesis-testing

## Summary

Non-parametric Mann–Whitney U test on null-segregated ion concentrations between two ROIs, used as parameter (b) of the rMSIKeyIon three-parameter ion-filtering algorithm for LDI-MS imaging data.

## When to use

Use when comparing ion concentrations between two regions of interest in an MSI peak matrix where undetected (null) peaks would otherwise invalidate parametric testing and must be discarded before the test.

## When NOT to use

- Input is already a filtered feature table rather than an rMSIproc peak matrix with pixel-level ion intensities
- The comparison involves more than two regions at once without pairwise comparisons, since the test is defined between two ROIs
- Data are continuous, fully detected, and normally distributed, where a parametric test on the raw concentrations would be preferable

## Inputs

- rMSIproc peak_matrix (e.g. Brain_peak_matrix.zip or peak_matrix.Rdata) with peak_matrix$intensity
- Clustering vector / ROI assignment (clustering.Rdata or kmeans cluster output)
- Percentile thresholds (demo: percentile = c(1,10,10))
- Zero concentration threshold (demo: zeroThreshold = 2.5e-4)

## Outputs

- R list with sub-list 'ions': up/down-regulated ions per ROI comparison
- R list with sub-list 'data': per-ion statistics (null ratio, Mann–Whitney U p-value, fold-change) per comparison
- results.Rdata file with the results list

## How to apply

Load an rMSIproc peak_matrix (e.g. Brain_peak_matrix.zip) and a clustering vector defining the ROIs (e.g. from kmeans(peak_matrix$intensity, centers = 7)$cluster). For each ion, first compute parameter (a), the percentage of pixels where the ion is undetected using zeroThreshold = 2.5e-4, then segregate null pixels from detected pixels; undetected signals are discarded from the histogram before the calculation of parameter (b) and the fold-change (c), because a high percentage of undetected MS ions undermines statistical test reliability. Run the Mann–Whitney U test on the null-segregated ion concentrations between the two ROIs via rMSIKeyIon::TestIonSelect, combining it with the null-ratio and median fold-change parameters through percentile-based discriminant thresholds (demo: percentile = c(1,10,10)). The output classifies ions as up- or down-regulated between the ROIs and is returned as an R list of two sub-lists, 'ions' and 'data', saved to results.Rdata.

## Related tools

- **rMSIKeyIon** (Implements TestIonSelect, which performs the null-segregated Mann–Whitney U test and combines it with the null-ratio and fold-change parameters) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix and visualizes the clustering and ions used as test input) — https://github.com/prafols/rMSIproc
- **RcppGSL** (Links the C++ code of rMSIKeyIon to R, requiring the GSL library)
- **R** (Runtime environment; kmeans is used to generate the clustering vector of ROIs)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Synthetic self-test: generate a synthetic peak matrix and two-ROI cluster vector in R and confirm TestIonSelect runs end-to-end returning correctly structured 'ions' and 'data' sub-lists
- Verify undetected pixels (intensity <= zeroThreshold = 2.5e-4) are excluded from the Mann–Whitney U computation per ion
- Check output structure matches the two-list format ('ions' and 'data') and persists to results.Rdata
- Confirm ion labels (up/down-regulated) are consistent with percentile discriminant thresholds c(1,10,10)

## Limitations

- High percentage of pixels with undetected MS ions is a difficulty for statistical test reliability, motivating the null-segregation step
- Package requires the GSL library (plus Rcpp/RcppGSL), which can complicate installation on Windows
- No discussion section is present in the article, so further limitations and reproducibility gaps cannot be extracted

## Evidence

- [abstract] Null peaks discarded before U test and fold-change: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [abstract] Three-parameter filter definition: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test, and (c) the ion concentration"
- [intro] Unreliable tests due to undetected ions: "Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels."
- [readme] Demo invocation with thresholds: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] Output structure of the package: "The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data belonging to each ion"
