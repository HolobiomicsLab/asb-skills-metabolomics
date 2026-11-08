---
name: zigzag-index-computation-from-derivative-variance
description: Use when LC/MS metabolomics EICs must be filtered before peak detection,
  especially when the mass chromatographic quality (MCQ) index fails on EICs with
  high background signals or only a single peak.
when_to_use_negative:
- When the goal is to evaluate already-detected chromatographic peaks with a metric
  other than the local zigzag index (e.g., sharpness or Gaussian similarity alone)
- When the EIC has no background or multi-peak noise and the MCQ index already gives
  a fair quality score
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Implement the zigzag index computation and apply it to EIC intensity vectors
- name: XCMS
  role: Used to produce the individual peak lists that define ground truth peaks for
    evaluating the filter
- name: MZmine
  role: Used to produce the individual peak lists that define ground truth peaks for
    evaluating the filter
- name: MAVEN
  role: Used to produce the individual peak lists that define ground truth peaks for
    evaluating the filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:deebc0f84ad1faf962fc5da882fc76a0778fa4a43f822fbaa3b8551e72297cf3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# zigzag-index-computation-from-derivative-variance

## Summary

Computes a robust EIC quality index from the sum of squared first-order-derivative terms of an extracted ion chromatogram, normalized by peak count and the squared apex EPI value, to flag noisy EICs with high background or only a single peak.

## When to use

Use when LC/MS metabolomics EICs must be filtered before peak detection, especially when the mass chromatographic quality (MCQ) index fails on EICs with high background signals or only a single peak.

## When NOT to use

- When the goal is to evaluate already-detected chromatographic peaks with a metric other than the local zigzag index (e.g., sharpness or Gaussian similarity alone)
- When the EIC has no background or multi-peak noise and the MCQ index already gives a fair quality score

## Inputs

- Extracted ion chromatogram (EIC) intensity vector I1…IN from an LC/MS metabolomics profile dataset
- Optionally, the apex intensity EPI and baseline level to enable baseline subtraction before normalization

## Outputs

- A scalar normalized zigzag index per EIC (range 0–4.0; lower = higher quality)
- A filter decision (accept/reject) per EIC when a cutoff threshold is applied

## How to apply

Take the EIC intensity series I1…IN and compute the first-order derivative terms (2·In − In−1 − In+1)² for each interior point, sum them, then normalize by N × EPI², where EPI is the baseline-subtracted apex intensity; normalize by the EIC dynamic range so values fall between 0 and 4.0, with lower values denoting higher EIC quality. Apply the same formulation to chromatographic peaks using the peak's left/right boundaries and apex intensity as the local zigzag index. Set a cutoff threshold on the normalized zigzag value to accept or reject the EIC, and combine it with other metrics (sharpness, peak significance level, SNR) because multi-metric filtering outperforms single metrics. Threshold selection is still an open challenge, so calibrate cutoffs empirically on known-good and known-bad EICs before applying them to the full dataset.

## Related tools

- **Matlab** (Implement the zigzag index computation and apply it to EIC intensity vectors)
- **XCMS** (Used to produce the individual peak lists that define ground truth peaks for evaluating the filter)
- **MZmine** (Used to produce the individual peak lists that define ground truth peaks for evaluating the filter)
- **MAVEN** (Used to produce the individual peak lists that define ground truth peaks for evaluating the filter)

## Examples

```
matlab -r "I = eic_intensities; D = (2*I(2:end-1) - I(1:end-2) - I(3:end)).^2; EPI = max(I) - baseline; zigzag = sum(D)/(length(I)*EPI^2)"
```

## Evaluation signals

- Zigzag values must fall within 0–4.0 after normalization by the EIC dynamic range
- EICs with high background signals or only a single peak should receive markedly higher zigzag values than clean EICs
- Applying the zigzag cutoff should preserve peaks that at least two of XCMS, MZmine, and MAVEN detect (ground-truth peaks)
- Filtering with the zigzag index alone should yield lower Precision/Recall/F-Score than a multi-metric combination that includes sharpness and peak significance level

## Limitations

- Setting an optimal cutoff threshold for each metric still is a challenge task
- The index is sensitive to the baseline level used to compute EPI; misestimated baseline inflates or deflates the normalized value
- Machine learning-based automatic peak filtering (e.g., SVM) is not yet implemented because a curated training set of good and bad peaks does not yet exist

## Evidence

- [abstract] Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
