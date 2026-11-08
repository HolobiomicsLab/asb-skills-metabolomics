---
name: lc-ms-noise-zigzag-assessment
description: Use when processing LC/MS profile data (e.g. Agilent HPLC–Q-TOF Premier)
  where extracted ion chromatograms must be quality-scored and filtered prior to CWT-based
  chromatographic peak detection, or when detected peaks need to be ranked by sharpness,
  peak significance level, or zigzag index to.
when_to_use_negative:
- When the EIC already has high background signals or contains only a single peak
  and the MCQ index alone is being used for selection/alignment — the MCQ index cannot
  fairly evaluate these; use the global zigzag index instead.
- When peaks are already ground truth (detected by at least two of XCMS, MZmine, and
  MAVEN) and no further quality filtering is needed.
- When the goal is a machine-learning classifier (e.g. SVM) for peak quality — the
  curated training set of good and bad representative peaks does not yet exist.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Matlab
  role: Hosts the four-module data processing program (acute EIC extraction, EIC quality
    evaluation/filtering, chromatographic peak detection, peak quality evaluation/filtering)
    and computes the six peak metrics and zigzag indices
- name: XCMS
  role: R-based peak detection tool used alongside MZmine and MAVEN to produce ground
    truth peaks for evaluating metric performance
- name: MZmine
  role: Java-based peak detection tool used alongside XCMS and MAVEN for ground truth
    peak definition
- name: MAVEN
  role: C++ machine-learning-based peak filtering tool used alongside XCMS and MZmine
    for ground truth peak definition
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:9443f6cbb415b8f90186060cd953d5eb7e7a1e0a2e4763056967cba590a71c23
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lc-ms-noise-zigzag-assessment

## Summary

Evaluates quality of extracted ion chromatograms (EICs) and detected chromatographic peaks in LC/MS metabolomics data using the MCQ index, a global EIC zigzag index derived from first-order derivatives, and six peak metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index), ranking them by F-Score to filter 'bad' EICs and peaks before downstream peak detection.

## When to use

Use when processing LC/MS profile data (e.g. Agilent HPLC–Q-TOF Premier) where extracted ion chromatograms must be quality-scored and filtered prior to CWT-based chromatographic peak detection, or when detected peaks need to be ranked by sharpness, peak significance level, or zigzag index to retain only good peaks for metabolomics feature lists.

## When NOT to use

- When the EIC already has high background signals or contains only a single peak and the MCQ index alone is being used for selection/alignment — the MCQ index cannot fairly evaluate these; use the global zigzag index instead.
- When peaks are already ground truth (detected by at least two of XCMS, MZmine, and MAVEN) and no further quality filtering is needed.
- When the goal is a machine-learning classifier (e.g. SVM) for peak quality — the curated training set of good and bad representative peaks does not yet exist.

## Inputs

- LC/MS profile dataset generated on an Agilent HPLC system interfaced with a quadrupole-time-of-flight (Q-TOF) Premier mass spectrometer
- Extracted ion chromatograms (EICs) with intensity-vs-retention-time profiles
- Detected chromatographic peak profiles (left and right boundary intensities, apex) from CWT-based peak detection
- Twelve representative peaks (six 'good', six 'bad') selected with expert consultation for threshold calibration

## Outputs

- EIC quality scores (MCQ index, global zigzag index) and filtered EIC list
- Per-peak values for sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local zigzag index
- Per-metric Recall, Precision, and F-Score curves at varying cutoff thresholds
- Filtered chromatographic peak list (good peaks) for downstream metabolomics analysis

## How to apply

Implement the four-module Matlab pipeline: acute EIC extraction, EIC quality evaluation/filtering, chromatographic peak detection, then peak quality evaluation/filtering. For EICs, compute the mass chromatographic quality (MCQ) index for alignment-type tasks and the global zigzag index from the EIC's first-order derivatives — the latter is robust when an EIC has high background signals or contains only a single peak. For detected peaks, compute six metrics: sharpness via the ratio-sum formula (equation 7), Gaussian similarity from the dot product of peak intensities and a Gaussian fit, SNR as the ratio of the CWT coefficient at a marker point to the 95% quantile of absolute CWT coefficients at scale 1, peak significance level as the ratio of mean intensity near the apex to mean intensity near the two boundaries, TPASR via equation 8, and the local zigzag index via the first-order derivative variance formula (equation 6). Vary each metric's cutoff threshold individually and in combination, and compute Recall, Precision, and F-Score to select thresholds; sharpness, peak significance level, and zigzag index outperform the others because LC/MS chromatographic peaks are inherently zigzag. Normalize each metric to [-1.0, 1.0] (subtract mean, divide by max) before clustering or ranking.

## Related tools

- **Matlab** (Hosts the four-module data processing program (acute EIC extraction, EIC quality evaluation/filtering, chromatographic peak detection, peak quality evaluation/filtering) and computes the six peak metrics and zigzag indices)
- **XCMS** (R-based peak detection tool used alongside MZmine and MAVEN to produce ground truth peaks for evaluating metric performance)
- **MZmine** (Java-based peak detection tool used alongside XCMS and MAVEN for ground truth peak definition)
- **MAVEN** (C++ machine-learning-based peak filtering tool used alongside XCMS and MZmine for ground truth peak definition)

## Examples

```
matlab -r "run_eic_peak_quality_pipeline('raw_profile.mzML', 'results_table.tsv')"
```

## Evaluation signals

- Sharpness, peak significance level, and zigzag index should yield the highest maximum F-Score when cutoffs are varied with EIC quality filtering bypassed — their F-Score curves should sit above those of Gaussian similarity, SNR, and TPASR.
- The global zigzag index should remain robust on EICs with high background signals or containing only a single peak, where the MCQ index fails.
- Per-peak metric values normalized to [-1.0, 1.0] should cluster the twelve representative peaks into the expected good/bad grouping.
- Recall, Precision, and F-Score must be reported for each metric at both individual and combined threshold settings.
- Ground truth peaks (detected by ≥2 of XCMS, MZmine, MAVEN) should be recovered in the filtered peak list.

## Limitations

- Setting an optimal cutoff threshold for each metric still is a challenge task.
- Machine learning-based automatic peak identification (e.g. SVM) is not yet implemented because a sufficient number of curated 'good' and 'bad' representative peaks for training does not yet exist.
- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak.
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository or license is provided.

## Evidence

- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [methods] which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering: "which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [intro] We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio: "We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
