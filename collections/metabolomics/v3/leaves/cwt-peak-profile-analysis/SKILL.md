---
name: cwt-peak-profile-analysis
description: Use when you have CWT-detected LC/MS chromatographic peaks and need to
  rank or compare peak quality evaluation metrics, especially when testing which metric
  distinguishes genuine signal peaks from noise or baseline artifacts.
when_to_use_negative:
- When peaks have already been filtered to a single 'good' set and no metric comparison
  is needed — the skill is designed for discriminating between good and bad peaks
  across multiple metrics.
- When the input is not chromatographic peak profile data from LC/MS (e.g. mass spectra
  or NMR data) — the zigzag-index rationale is specific to LC/MS peak morphology.
- When only a single metric's cutoff needs tuning — the skill's core value is the
  head-to-head comparison of six metrics, not threshold optimization in isolation.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Matlab
  role: 'Primary implementation environment: the paper''s own data processing program
    (four sequential modules including peak quality evaluation and filtering) is developed
    in Matlab; metric computation, normalization, clustering, and F-Score ranking
    are all performed here.'
- name: XCMS
  role: Used (alongside MZmine and MAVEN) to produce individual peak lists on the
    same UPLC/MS dataset for ground truth peak definition — peaks detected by at least
    two tools were treated as ground truth, providing the reference set against which
    metric performance is judged.
- name: MZmine
  role: Source of one of the three independent peak lists used for ground truth definition;
    peaks found by ≥2 of XCMS/MZmine/MAVEN are ground truth peaks.
- name: MAVEN
  role: Source of one of the three independent peak lists used for ground truth definition;
    its complex machine learning-based peak filtering method also illustrates a contrasting
    peak quality strategy.
- name: Agilent HPLC system
  role: Instrument used to generate the real LC/MS profile dataset whose chromatographic
    peaks are evaluated by the six metrics.
- name: Q-TOF Premier mass spectrometer
  role: Mass spectrometer coupled to the Agilent HPLC, producing the profile-mode
    LC/MS data from which peaks are extracted and scored.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:9cefcf5cbf629e7bb2a03d5277eec81d0fae34562a438ab038d078e3959049cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent HPLC system
  - MAVEN
  - MZmine
  - Matlab
  - Q-TOF Premier mass spectrometer
  - XCMS
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cwt-peak-profile-analysis

## Summary

Evaluate the quality of LC/MS chromatographic peaks extracted via continuous wavelet transform (CWT) by computing six metrics — sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and a local peak-related zigzag index — and comparing their discriminative power between 'good' and 'bad' peaks.

## When to use

Use when you have CWT-detected LC/MS chromatographic peaks and need to rank or compare peak quality evaluation metrics, especially when testing which metric best distinguishes genuine signal peaks from noise or baseline artifacts.

## When NOT to use

- When peaks have already been filtered to a single 'good' set and no metric comparison is needed — the skill is designed for discriminating between good and bad peaks across multiple metrics.
- When the input is not chromatographic peak profile data from LC/MS (e.g. mass spectra or NMR data) — the zigzag-index rationale is specific to LC/MS peak morphology.
- When only a single metric's cutoff needs tuning — the skill's core value is the head-to-head comparison of six metrics, not threshold optimization in isolation.

## Inputs

- Twelve representative LC/MS chromatographic peaks (six 'good', six 'bad') selected with expert consultation from CWT-based peak detection output
- Peak profile intensity arrays between left and right boundaries
- CWT coefficients at the marker point and at scale 1 (for SNR)

## Outputs

- Per-peak values for sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local zigzag index
- Clustering group assignment of the twelve peaks based on all six normalized metrics
- Per-metric F-Score, Recall, and Precision ranking across varied cutoff thresholds (results_table.tsv)

## How to apply

Load representative chromatographic peaks (e.g. the paper's twelve expert-selected peaks — six 'good', six 'bad') into Matlab. Compute sharpness using the ratio-sum formula over peak profile intensities between left and right boundaries (equation 7); Gaussian similarity from the dot product of detected peak intensities against a Gaussian curve fit; SNR as the ratio of the CWT coefficient at the marker point to the 95% quantile of absolute CWT coefficients at scale 1; peak significance level as the ratio of mean intensity near the apex to mean intensity near the two boundaries; TPASR as the absolute difference between triangle peak area and real peak area, normalized by triangle peak area (equation 8); and the local zigzag index via the first-order derivative variance formula (equation 6) over the local peak profile. Normalize each metric to [-1.0, 1.0] by subtracting the mean and dividing by the maximum, then cluster the twelve peaks using all six normalized values. Finally, vary each metric's cutoff threshold systematically (with EIC quality filtering bypassed) and record Recall, Precision, and F-Score; the metrics whose F-Score maxima are highest — sharpness, peak significance level, and zigzag index — should match the paper's finding that these outperform Gaussian similarity, SNR, and TPASR because of the zigzag nature of LC/MS chromatographic peaks.

## Related tools

- **Matlab** (Primary implementation environment: the paper's own data processing program (four sequential modules including peak quality evaluation and filtering) is developed in Matlab; metric computation, normalization, clustering, and F-Score ranking are all performed here.)
- **XCMS** (Used (alongside MZmine and MAVEN) to produce individual peak lists on the same UPLC/MS dataset for ground truth peak definition — peaks detected by at least two tools were treated as ground truth, providing the reference set against which metric performance is judged.)
- **MZmine** (Source of one of the three independent peak lists used for ground truth definition; peaks found by ≥2 of XCMS/MZmine/MAVEN are ground truth peaks.)
- **MAVEN** (Source of one of the three independent peak lists used for ground truth definition; its complex machine learning-based peak filtering method also illustrates a contrasting peak quality strategy.)
- **Agilent HPLC system** (Instrument used to generate the real LC/MS profile dataset whose chromatographic peaks are evaluated by the six metrics.)
- **Q-TOF Premier mass spectrometer** (Mass spectrometer coupled to the Agilent HPLC, producing the profile-mode LC/MS data from which peaks are extracted and scored.)

## Evaluation signals

- The metrics with the highest maximum F-Score must be sharpness, peak significance level, and the zigzag index, matching the paper's reported finding.
- F-Score curves for sharpness, peak significance level, and zigzag index should sit higher in magnitude than those for Gaussian similarity, SNR, and TPASR.
- All six normalized metric values must fall within [-1.0, 1.0].
- Clustering of the twelve peaks must separate the six 'good' peaks from the six 'bad' peaks using the six normalized metrics.
- Per-peak metric values, cluster assignments, and F-Score rankings must be persisted to results_table.tsv for auditability.

## Limitations

- Optimal cutoff thresholds for each quality metric remain an unsolved challenge; no principled threshold-selection procedure is provided.
- A machine learning-based automatic peak identification approach (e.g. SVM) is not yet implemented, and no curated training set of sufficient 'good' and 'bad' representative peaks yet exists.
- The mass chromatographic quality (MCQ) index cannot fairly evaluate EICs with high background signals or those containing only a single peak — a gap the global zigzag index is designed to fill, but it does not replace all metric shortcomings.
- The LC/MS profile dataset and Matlab scripts are available only upon request and no public repository is provided, so reproduction depends on obtaining the authors' code.
- No license is specified for the software ('License: None for usage.').

## Evidence

- [abstract] Sharpness, peak significance level, and zigzag index outperform the other three metrics (Gaussian similarity, SNR, and TPASR) due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] Metric computation formulas: [evidence span withheld — anchor status: not_found]
- [methods] SNR definition: "SNR as the ratio of the CWT coefficient at a marker point to the 95% quantile of the absolute CWT coefficient at scale 1."
- [methods] TPASR definition: [evidence span withheld — anchor status: not_found]
- [methods] zigzag index definition: [evidence span withheld — anchor status: not_found]
- [methods] Normalization and clustering: [evidence span withheld — anchor status: not_found]
- [methods] F-Score ranking procedure: [evidence span withheld — anchor status: not_found]
- [methods] Ground truth peak definition: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [methods] Tool comparison scope: "XCMS was developed in R and configured with two options, 'MatchedFilter' and 'centWave', to detect meaningful chromatographic peaks."
- [methods] MAVEN's contrasting approach: "MAVEN was developed in C++ and equipped with a complex, machine learning-based peak filtering method."
- [methods] Instrument context: "a real LC/MS profile dataset was generated using an Agilent HPLC system interfaced with a quadrupole-time-of-flight (Q-TOF) Premier mass spectrometer"
- [discussion] Cutoff threshold challenge: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [discussion] ML limitation: "This requires compiling and curating a sufficient number of "good" and "bad" representative peaks for training samples."
- [discussion] Data availability caveat: "The LC/MS profile dataset and Matlab scripts are available upon request."
