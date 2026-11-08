---
name: lc-ms-peak-sharpness-and-significance-evaluation
description: Use when you have candidate chromatographic peaks detected from LC/MS
  EICs (e.
when_to_use_negative:
- The EIC itself is of unknown or poor quality — the article recommends first applying
  EIC quality evaluation and filtering (module 2, using the MCQ index or the global
  zigzag index) before running peak-level metrics, and to bypass EIC filtering only
  when specifically benchmarking peak metrics against a fixed ground truth.
- You are trying to select or align analyte components from EICs containing high background
  signals or only a single peak — the MCQ index is not suited for those cases; use
  the EIC-related global zigzag index instead.
- 'You need a fully automated, machine-learning-based peak filter (e.g., SVM): the
  article notes this requires compiling and curating a sufficient number of ''good''
  and ''bad'' representative peaks for training samples, which does not yet exist
  in the provided context.'
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Matlab
  role: 'Platform in which the authors'' own data-processing program (four sequential
    modules: acute EIC extraction, EIC quality evaluation and filtering, chromatographic
    peak detection, peak quality evaluation and filtering) is implemented; Matlab
    scripts execute the sharpness, Gaussian similarity, SNR, peak significance level,
    TPASR, and zigzag index computations and the F-Score benchmarking.'
- name: XCMS
  role: One of three open-source tools used to analyze the same UPLC/MS dataset and
    produce individual peak lists that feed the ground-truth peak definition against
    which metric performance is evaluated.
- name: MZmine
  role: One of three open-source tools (developed in Java with the centroidPicker
    algorithm) used to produce individual peak lists for ground-truth peak definition.
- name: MAVEN
  role: One of three open-source tools (developed in C++ with a complex, machine learning-based
    peak filtering method) used to produce individual peak lists for ground-truth
    peak definition.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:f20e50866b6d87d2f54d28a1b956c09bb77ccf823f12ecc49e2fd263a70dd868
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lc-ms-peak-sharpness-and-significance-evaluation

## Summary

Computes and benchmarks chromatographic peak quality metrics — sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and the local zigzag index — on LC/MS extracted-ion peaks, identifying which metrics best separate good from bad peaks. Applies to the paper's fourth pipeline module, peak quality evaluation and filtering, prior to downstream processing.

## When to use

Use when you have candidate chromatographic peaks detected from LC/MS EICs (e.g., by XCMS, MZmine, or MAVEN CWT-based detection) and need to score, rank, or filter them by quality using shape- and signal-based metrics, especially when you must determine which metrics actually discriminate good from bad peaks.

## When NOT to use

- The EIC itself is of unknown or poor quality — the article recommends first applying EIC quality evaluation and filtering (module 2, using the MCQ index or the global zigzag index) before running peak-level metrics, and to bypass EIC filtering only when specifically benchmarking peak metrics against a fixed ground truth.
- You are trying to select or align analyte components from EICs containing high background signals or only a single peak — the MCQ index is not suited for those cases; use the EIC-related global zigzag index instead.
- You need a fully automated, machine-learning-based peak filter (e.g., SVM): the article notes this requires compiling and curating a sufficient number of 'good' and 'bad' representative peaks for training samples, which does not yet exist in the provided context.

## Inputs

- Detected chromatographic peaks from LC/MS EICs (peak apex and boundary intensities, e.g., from XCMS, MZmine, or MAVEN output on a Q-TOF Premier profile dataset)
- Raw EIC intensity profiles for each candidate peak (left and right boundary intensities plus apex-region intensities)
- Optional: a labeled set of 'good' and 'bad' representative peaks for training a machine-learning-based filter (not yet available per the article)

## Outputs

- Per-peak metric values table (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, zigzag index) saved to results_table.tsv
- Clustering group assignment for each peak based on the six normalized metric values
- Per-metric Recall, Precision, and F-Score curves obtained by varying each metric's cutoff threshold, yielding a ranking of metrics by maximum F-Score

## How to apply

Load candidate peaks (six 'good' and six 'bad' representatives are sufficient for a controlled comparison; here twelve peaks selected with expert consultation from CWT-based peak detection output) into Matlab and compute each metric over the peak profile between left and right boundaries: sharpness via the ratio-sum formula (equation 7); Gaussian similarity via the dot product of detected peak intensities against Gaussian curve fitting; SNR as the ratio of the CWT coefficient at a marker point to the 95% quantile of absolute CWT coefficients at scale 1; peak significance level as the ratio of mean intensity near the apex to mean intensity near the two boundaries; TPASR as the absolute difference between triangle peak area and real peak area normalized by triangle peak area (equation 8); and the local zigzag index via the first-order-derivative variance formula (equation 6). Normalize each metric to [-1.0, 1.0] by subtracting the mean and dividing by the maximum, then cluster the peaks on the six normalized metric values and vary each metric's cutoff threshold systematically (with EIC quality filtering bypassed) to record Recall, Precision, and F-Score per metric. The expected outcome is that sharpness, peak significance level, and the zigzag index produce the highest F-Score maxima — a direct consequence of the zigzag nature of LC/MS chromatographic peaks — while Gaussian similarity, SNR, and TPASR underperform; combine several of the top metrics rather than relying on any single one, since combination is more efficient than individual metrics for peak quality evaluation.

## Related tools

- **Matlab** (Platform in which the authors' own data-processing program (four sequential modules: acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering) is implemented; Matlab scripts execute the sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and zigzag index computations and the F-Score benchmarking.)
- **XCMS** (One of three open-source tools used to analyze the same UPLC/MS dataset and produce individual peak lists that feed the ground-truth peak definition against which metric performance is evaluated.)
- **MZmine** (One of three open-source tools (developed in Java with the centroidPicker algorithm) used to produce individual peak lists for ground-truth peak definition.)
- **MAVEN** (One of three open-source tools (developed in C++ with a complex, machine learning-based peak filtering method) used to produce individual peak lists for ground-truth peak definition.)

## Examples

```
matlab -batch "run('compute_peak_metrics.m')"  % computes sharpness, Gaussian similarity, SNR, peak significance level, TPASR, zigzag index; writes results_table.tsv
```

## Evaluation signals

- Metrics with the highest F-Score maxima (varying each cutoff threshold with EIC quality filtering bypassed) must be sharpness, peak significance level, and the zigzag index, outperforming Gaussian similarity, SNR, and TPASR — matching the article's reported finding.
- All six metric values must be normalized to the range [-1.0, 1.0] (subtract mean, divide by maximum) before clustering; verify no value falls outside this interval.
- On the twelve-peak benchmark (six 'good', six 'bad'), clustering on the normalized metrics should separate the two groups; inspect the cluster assignment against the expert-consulted labels.
- Per-metric Recall, Precision, and F-Score must be recorded for every systematic cutoff threshold swept; verify that the ranking is based on the maximum F-Score magnitude per metric.
- Peak significance level must be computed as the ratio of mean intensity near the apex to mean intensity near the two boundaries, and TPASR as |triangle peak area − real peak area| / triangle peak area (equation 8); recompute on a subset of peaks to confirm formula fidelity.

## Limitations

- Setting an optimal cutoff threshold for each metric still is a challenge task; no universal threshold is provided.
- Machine-learning-based automatic peak identification (e.g., SVM) is not yet implemented and requires compiling and curating a sufficient number of 'good' and 'bad' representative peaks for training samples, which does not yet exist.
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository is provided, and the license is 'None for usage'.
- Performance conclusions rest on peaks detected by at least two of XCMS, MZmine, and MAVEN (ground truth definition); results may differ with a different ground-truth construction.

## Evidence

- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: "we developed our own data processing program in Matlab, which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection,"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [abstract] Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
