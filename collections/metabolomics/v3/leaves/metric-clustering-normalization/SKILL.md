---
name: metric-clustering-normalization
description: Use when multiple peak quality metrics computed on curated good/bad LC/MS
  peaks must be brought to a common scale, clustered across representative peaks,
  and ranked by maximum F-Score to identify which metrics are most discriminative.
when_to_use_negative:
- Evaluating whole extracted ion chromatograms (EICs) rather than individual detected
  peaks — the paper handles that with the MCQ index and the EIC-related global zigzag
  index, not these six peak metrics
- EICs with high background signals or containing only a single peak — the MCQ index
  cannot fairly evaluate these scenarios, so peak-metric normalization on such inputs
  is not meaningful
- Input is already a normalized feature matrix or clustering result; re-normalizing
  to [-1, 1] by mean-subtraction/max-division would distort existing scales
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Implementation platform for the custom data processing program that computes
    the six metrics, performs normalization, and runs the clustering analysis on the
    twelve peaks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:d0fd659d1c4d8fd6cab4567e0357fd4cbdcb1431c591d1093317f01ea02c9eef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metric-clustering-normalization

## Summary

Normalizes chromatographic peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) to a common [-1, 1] range and clusters/ranks them to determine which metrics best discriminate good from bad LC/MS peaks. Grounded in the BMC Bioinformatics 2014 evaluation showing sharpness, peak significance level, and the zigzag index outperform the others.

## When to use

Use when multiple peak quality metrics computed on curated good/bad LC/MS peaks must be brought to a common scale, clustered across representative peaks, and ranked by maximum F-Score to identify which metrics are most discriminative.

## When NOT to use

- Evaluating whole extracted ion chromatograms (EICs) rather than individual detected peaks — the paper handles that with the MCQ index and the EIC-related global zigzag index, not these six peak metrics
- EICs with high background signals or containing only a single peak — the MCQ index cannot fairly evaluate these scenarios, so peak-metric normalization on such inputs is not meaningful
- Input is already a normalized feature matrix or clustering result; re-normalizing to [-1, 1] by mean-subtraction/max-division would distort existing scales

## Inputs

- Twelve representative chromatographic peaks (six 'good' and six 'bad') selected with expert consultation from CWT-based peak detection output
- Per-peak profile intensity arrays with left and right peak boundaries
- CWT coefficient matrices (scale 1 coefficients for SNR computation)

## Outputs

- Per-peak table of six metric values normalized to [-1.0, 1.0]
- Clustering group assignment for the twelve peaks
- Per-metric Recall, Precision, and maximum F-Score ranking (results_table.tsv)

## How to apply

Compute the six metrics per peak in Matlab: sharpness as the ratio-sum formula (equation 7) over intensities between the peak's left and right boundaries; Gaussian similarity from the dot product of detected peak intensities with Gaussian curve fitting; SNR as the CWT coefficient at the marker point divided by the 95% quantile of absolute CWT coefficients at scale 1; peak significance level as the ratio of mean intensity near the apex to mean intensity near the two boundaries; TPASR as the absolute difference between triangle peak area and real peak area normalized by triangle peak area (equation 8); and the local zigzag index via the first-order derivative variance formula (equation 6) over the local peak profile. Normalize each metric to [-1.0, 1.0] by subtracting the mean and dividing by the maximum so that metrics on incomparable scales contribute equally. Cluster the twelve peaks using all six normalized metric values jointly, then vary each metric's cutoff threshold systematically with EIC quality filtering bypassed and record Recall, Precision, and F-Score per metric. Rank metrics by the magnitude of their maximum F-Score; the rationale is that combining several quality metrics is more efficient than any individual metric, and the ranking reveals which metrics capture the zigzag nature of LC/MS chromatographic peaks.

## Related tools

- **Matlab** (Implementation platform for the custom data processing program that computes the six metrics, performs normalization, and runs the clustering analysis on the twelve peaks)

## Examples

```
% Matlab: normalize metric matrix M (12 peaks x 6 metrics) to [-1,1] then cluster
Z = (M - mean(M)) ./ max(abs(M)); idx = kmeans(Z, 2);
```

## Evaluation signals

- All normalized metric values fall within [-1.0, 1.0] after mean subtraction and max division
- Clustering on the six normalized metrics separates the six 'good' peaks from the six 'bad' peaks into distinct groups
- F-Score curves for sharpness, peak significance level, and zigzag index reach higher maxima than those for Gaussian similarity, SNR, and TPASR when cutoffs are varied with EIC quality filtering bypassed
- Recall, Precision, and F-Score are recorded for every metric at each varied cutoff threshold, not only at the maximum
- Combined use of several metrics yields better peak quality evaluation than any single metric, consistent with the paper's conclusion

## Limitations

- Setting an optimal cutoff threshold for each metric still is a challenge task — the ranking uses maximum F-Score but does not yield deployable thresholds
- Machine learning-based automatic peak identification (e.g., SVM) requires a curated training set of 'good' and 'bad' representative peaks that does not yet exist
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository or license for usage is provided, limiting reproducibility

## Evidence

- [abstract] Peak metric ranking rationale: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] Combination beats individual metrics: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] Matlab processing program: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [discussion] Cutoff threshold challenge: "setting an optimal cutoff threshold for each metric still is a challenge task"
