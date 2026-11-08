---
name: null-peak-zero-threshold-filtering
description: Use when comparing ion intensities between MSI regions-of-interest (e.g.,
  k-means clusters of LDI-MS liver or brain images) and the peak matrix contains many
  ions undetected in a high percentage of pixels, which otherwise undermines the Mann-Whitney
  U test and fold-change reliability.
when_to_use_negative:
- The peak matrix is already a dense feature table with essentially no undetected
  peaks, so the null-peak percentage filter adds no discriminating power.
- You only want to inspect precomputed demo results, in which case loading results.Rdata
  directly is sufficient and re-running TestIonSelect is unnecessary.
- The analysis goal is image segmentation itself (k-means clustering) rather than
  statistical ion selection between already-defined regions.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: rMSIKeyIon
  role: Implements TestIonSelect with the zeroThreshold parameter to discard undetected
    MS signals before computing the Mann-Whitney U and fold-change parameters.
  repo: LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the peak matrix from LDI-MSI images whose null peaks are then filtered.
  repo: prafols/rMSIproc
- name: kmeans
  role: Produces the clustering vector (ROIs) over which up/down-regulated ions are
    statistically compared after null-peak filtering.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:714c9fc27358d27b26e48372d6a464c56e65f8c3d10ca45b3770fa2045af2f17
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Null-Peak Zero-Threshold Filtering

## Summary

A statistical ion-filtering refinement from the rMSIKeyIon R package that treats undetected MS signals ('null peaks') explicitly, using a zeroThreshold (default 2.5e-4) so ions missing in a high percentage of pixels do not corrupt p-value/fold-change comparisons between image regions.

## When to use

Use when comparing ion intensities between MSI regions-of-interest (e.g., k-means clusters of LDI-MS liver or brain images) and the peak matrix contains many ions undetected in a high percentage of pixels, which otherwise undermines the Mann-Whitney U test and fold-change reliability.

## When NOT to use

- The peak matrix is already a dense feature table with essentially no undetected peaks, so the null-peak percentage filter adds no discriminating power.
- You only want to inspect precomputed demo results, in which case loading results.Rdata directly is sufficient and re-running TestIonSelect is unnecessary.
- The analysis goal is image segmentation itself (k-means clustering) rather than statistical ion selection between already-defined regions.

## Inputs

- rMSIproc peak matrix (peak_matrix from rMSIproc::LoadPeakMatrix of a .zip, or peak_matrix.Rdata)
- clustering_vector (e.g., from kmeans(peak_matrix$intensity, centers = 6)$cluster) or ROI annotations
- percentile parameter vector (article/README value: c(1,10,10))
- zeroThreshold value (article/README value: 2.5e-4)

## Outputs

- results list with element 'ions': up/down-regulated ions per cluster/ROI comparison
- results list with element 'data': statistical data (null-peak percentage, Mann-Whitney U, fold-change) for each ion at each comparison
- results.Rdata file storing the TestIonSelect output

## How to apply

Load the peak matrix (rMSIproc::LoadPeakMatrix of a Brain_peak_matrix.zip or base load() of peak_matrix.Rdata) and a clustering vector from kmeans on peak_matrix$intensity. Call rMSIKeyIon::TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4; the zeroThreshold defines the intensity cut below which an ion is considered undetected in a pixel. The algorithm computes parameter (a), the percentage of pixels in which a particular ion is not detected, and discards those null-peak pixels from the histograms used to derive the Mann-Whitney U ion concentration test (b) and the ion concentration fold-change (c). The returned 'results' list contains 'ions' (up/down-regulated ions per comparison) and 'data' (statistics for each ion), which can be saved as results.Rdata. The rationale is that the classical p-value/FC method is unreliable when many pixels lack signal, so the null-peak percentage is treated as a third, explicit filter.

## Related tools

- **rMSIKeyIon** (Implements TestIonSelect with the zeroThreshold parameter to discard undetected MS signals before computing the Mann-Whitney U and fold-change parameters.) — LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the peak matrix from LDI-MSI images whose null peaks are then filtered.) — prafols/rMSIproc
- **kmeans** (Produces the clustering vector (ROIs) over which up/down-regulated ions are statistically compared after null-peak filtering.)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- The TestIonSelect call completes without GSL/C++ linking errors and returns a list with exactly two elements named 'ions' and 'data'.
- Ions whose undetected-pixel percentage exceeds the zeroThreshold-derived null criterion are excluded from the up/down-regulated lists rather than being called significant by spurious fold-changes.
- Re-running TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4 on the same peak matrix and clustering_vector reproduces the same 'ions' list (deterministic output).
- For the THS liver study, the filtered output shows absolutely up-regulated ions in cluster C2 within the lipid mass range, which map to triacylglycerols and phosphatidylcholines in Table S1.
- Saved results.Rdata can be re-loaded with base load() and its structure matches the documented two-list ('ions' and 'data') schema.

## Limitations

- An elevated number of undetected MS ions in a high percentage of pixels is itself a difficulty for the reliability of statistical tests; the zero-threshold method mitigates but does not eliminate this.
- The method depends on a chosen zeroThreshold (2.5e-4 in the article/README); inappropriate thresholds will misclassify detected vs. undetected peaks.
- No discussion/changelog text was available, so limitations, missing artifacts, and reproducibility gaps beyond those stated in the abstract could not be verified.

## Evidence

- [abstract] Null peaks discarded before statistics: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [readme] Third method for undetected peaks: "It uses the classical p-value/ FC method and introduces a third novel method to treat undetected MS peaks more accurately."
- [readme] Concrete parameter values: "percentile = c(1,10,10),
                                    zeroThreshold = 2.5e-4)"
- [abstract] Three-parameter filter definition: "by using a combination of three parameters: (a) the percentage of pixels in which a particular ion is not detected, (b) the Mann-Whitney U ion concentration test, and (c) the ion concentration"
- [readme] Output list schema: "The first one, called "ions" contains the up/down-regulated ions
found at each comparison. The second one, called "data" contains all the data belonging to each ion at each"
