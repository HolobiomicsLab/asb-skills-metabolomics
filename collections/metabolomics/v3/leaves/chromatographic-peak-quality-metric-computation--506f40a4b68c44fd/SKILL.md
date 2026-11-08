---
name: chromatographic-peak-quality-metric-computation
description: Use when LC/MS chromatographic peaks have been detected on extracted
  ion chromatograms (EICs) and you need per-peak numeric quality scores to filter
  'bad' peaks before downstream metabolomics processing.
when_to_use_negative:
- Input is already a filtered list of peaks with known-good quality (e.g. after SVM-based
  peak identification) — the metric computation would be redundant
- You need to evaluate the quality of extracted ion chromatograms (EICs) themselves
  rather than detected peaks — the paper's MCQ index and EIC-related global zigzag
  index address that separate problem
- The data are not LC/MS metabolomics chromatograms (e.g. GC/MS or non-chromatographic
  spectra), where the zigzag-nature rationale and Gaussian-shape assumptions may not
  hold
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Host language for the paper's own four-module data processing program, whose
    fourth module performs peak quality evaluation and filtering; all six metric formulas
    are implemented here
- name: XCMS
  role: One of the three open-source peak-detection tools used to build the ground
    truth peak list against which metric-based filtering is evaluated
- name: MZmine
  role: Second open-source peak-detection tool contributing to the ground truth peak
    list (matched with 0.05 Da m/z and 5 s RT tolerance)
- name: MAVEN
  role: Third open-source peak-detection tool contributing to the ground truth peak
    list (matched with 0.05 Da m/z and 5 s RT tolerance)
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:d5140cd272f0acfc6d6ac13825b5465a8f2d961f809138eb24c2d41ae6bd6a03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chromatographic-peak-quality-metric-computation

## Summary

Computes and compares the six chromatographic peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and the local zigzag index) defined by Zhang & Zhao for LC/MS-based metabolomics peaks, so that good and bad detected peaks can be distinguished and filtered.

## When to use

Use when LC/MS chromatographic peaks have been detected on extracted ion chromatograms (EICs) and you need per-peak numeric quality scores to filter 'bad' peaks before downstream metabolomics processing.

## When NOT to use

- Input is already a filtered list of peaks with known-good quality (e.g. after SVM-based peak identification) — the metric computation would be redundant
- You need to evaluate the quality of extracted ion chromatograms (EICs) themselves rather than detected peaks — the paper's MCQ index and EIC-related global zigzag index address that separate problem
- The data are not LC/MS metabolomics chromatograms (e.g. GC/MS or non-chromatographic spectra), where the zigzag-nature rationale and Gaussian-shape assumptions may not hold

## Inputs

- Detected chromatographic peak list with per-peak intensity vectors, apex index, and left/right peak boundaries
- EIC profiles (from LC/MS profile data, e.g. Agilent HPLC–Q-TOF Premier) underlying each peak
- Ground truth peak list (defined as peaks detected by at least two of XCMS, MZmine, and MAVEN, matched with 0.05 Da m/z tolerance and 5 seconds RT tolerance) for performance evaluation
- Optional set of representative 'good' and 'bad' peaks selected with expert consultation for case-study-based evaluation

## Outputs

- Per-peak table of six metric values: sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local zigzag index
- Recall, Precision, and F-Score per metric (when cutoff thresholds are varied) and for the combined-metric evaluation
- Clustering/group assignments when metric values are normalized to [-1.0, 1.0] and clustered on representative peaks

## How to apply

For each detected peak with an intensity vector, apex index, and left/right boundaries, compute sharpness as the sum of relative upward increments from the left boundary to the apex plus relative downward decrements from the apex to the right boundary (equation 7); Gaussian similarity as the dot product between the detected peak intensities and a fitted classical or modified Gaussian curve; SNR in the wavelet domain as the ratio of the CWT coefficient at a marker point to the 95% quantile of the absolute CWT coefficients at scale 1; peak significance level as the ratio of mean intensity near the apex to mean intensity near the two boundaries; TPASR as |TPA − RPA| / TPA with TPA = 0.5 × peak width × apex intensity and RPA the summed raw peak area between the boundaries (equation 8); and the local zigzag index by normalizing the sum of squared second differences (2·Iₙ − Iₙ₋₁ − Iₙ₊₁)² by N × EPI², where EPI is the baseline-subtracted apex intensity (equations 1–6, dynamic range 0–4.0, lower = higher quality). The paper implements these in a Matlab data processing program whose fourth module is 'peak quality evaluation and filtering', but the formulas are metric definitions independent of the host language. Apply the metrics to a peak list (optionally on twelve representative peaks — six 'good' and six 'bad' — for case studies, or the full dataset with thresholds varied individually and in combination) and record Recall, Precision, and F-Score against a ground truth list to evaluate performance.

## Related tools

- **Matlab** (Host language for the paper's own four-module data processing program, whose fourth module performs peak quality evaluation and filtering; all six metric formulas are implemented here)
- **XCMS** (One of the three open-source peak-detection tools used to build the ground truth peak list against which metric-based filtering is evaluated)
- **MZmine** (Second open-source peak-detection tool contributing to the ground truth peak list (matched with 0.05 Da m/z and 5 s RT tolerance))
- **MAVEN** (Third open-source peak-detection tool contributing to the ground truth peak list (matched with 0.05 Da m/z and 5 s RT tolerance))

## Examples

```
matlab -batch "run_peak_quality_metrics('peaks.csv', 'ground_truth.csv')"
```

## Evaluation signals

- Smooth Gaussian-like peaks must score high on sharpness, Gaussian similarity, SNR, and peak significance level and low on TPASR and zigzag index
- Zigzag-shaped peaks must receive markedly higher local zigzag index values (range 0–4.0, lower = better) than smooth peaks
- Varying a single metric's cutoff with EIC quality filtering bypassed, sharpness, peak significance level, and zigzag index must yield higher maximum F-Score magnitudes than Gaussian similarity, SNR, and TPASR
- The maximum single-metric F-Score must remain smaller than 0.6
- Combining all six metrics at the joint thresholds (Sharpness_Th = 2.0, Gaussian_Similarity_Th = 0.6, SNR_Th = 1.3, Peak_Significance_Th = 1.2, TPASR_Th = 0.8, Zigzag_Th = 0.9) must yield Recall = 0.7076, Precision = 0.6186, F-Score = 0.6601, exceeding every single-metric F-Score

## Limitations

- Setting an optimal cutoff threshold for each metric remains an unsolved challenge
- Machine learning-based automatic peak identification (e.g. SVM) is not yet implemented and would require curating a sufficient training set of 'good' and 'bad' representative peaks that does not yet exist
- The local zigzag index has a bounded dynamic range of 0–4.0, so values must be interpreted with that scale in mind (lower is better)
- Metric formulas assume the zigzag nature of LC/MS chromatographic peaks; their discriminative power may not transfer to other peak shapes or other '-omics' domains

## Evidence

- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering: "we developed our own data processing program in Matlab, which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection,"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
