---
name: first-order-derivative-zigzag-quantification
description: Use when extracted EICs from LC/MS-based metabolomics data must be scored
  for quality before downstream chromatographic peak detection, especially for EICs
  with high background signals or containing only a single peak.
when_to_use_negative:
- When the goal is selecting or aligning analyte components rather than quality evaluation
  — the article states the MCQ index, not the zigzag index, is suited for that task
- When a single metric is expected to give a definitive good/bad verdict for chromatographic
  peaks — the article recommends combining several peak quality metrics instead
- When an optimal cutoff threshold for each metric is required as a settled value
  — the article notes that setting an optimal cutoff threshold for each metric still
  is a challenge task
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Programming language in which the authors' four-module data processing program
    (including EIC and peak quality evaluation/filtering) was implemented
- name: XCMS
  role: Open-source tool used to produce peak lists for ground truth definition in
    comprehensive evaluation of the metrics
- name: MZmine
  role: Open-source tool used to produce peak lists for ground truth definition in
    comprehensive evaluation of the metrics
- name: MAVEN
  role: Open-source tool used to produce peak lists for ground truth definition in
    comprehensive evaluation of the metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:8c382c0493f9519589c5ef7bc5d4cfc7d6721d43620203806e54f8531cb02718
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# first-order-derivative-zigzag-quantification

## Summary

Computes an EIC-related global zigzag index from the first-order derivatives of an extracted ion chromatogram (EIC), yielding a value in [0, 4.0] that robustly quantifies signal jaggedness for high-background or single-peak EICs where the mass chromatographic quality (MCQ) index fails.

## When to use

Use when extracted EICs from LC/MS-based metabolomics data must be scored for quality before downstream chromatographic peak detection, especially for EICs with high background signals or containing only a single peak.

## When NOT to use

- When the goal is selecting or aligning analyte components rather than quality evaluation — the article states the MCQ index, not the zigzag index, is suited for that task
- When a single metric is expected to give a definitive good/bad verdict for chromatographic peaks — the article recommends combining several peak quality metrics instead
- When an optimal cutoff threshold for each metric is required as a settled value — the article notes that setting an optimal cutoff threshold for each metric still is a challenge task

## Inputs

- Extracted ion chromatogram (EIC) intensity profiles from LC/MS data (e.g. generated using an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer)
- Chromatographic peak intensity profiles after peak detection

## Outputs

- Per-EIC global zigzag index value in the range [0, 4.0]
- Per-peak zigzag index (one of six peak quality metrics: sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag)
- Filtered EIC/peak lists in which 'bad' EICs or peaks above a chosen cutoff threshold have been removed

## How to apply

Take each EIC's intensity profile (e.g. representative EIC_IDs 27, 40, 90, 134, 154, 167 from the 611 extracted EICs) and compute its first-order derivatives across neighboring data points. Sum the squared quantities (2*In - In-1 - In+1) over the EIC and normalize by N times the squared effective peak intensity (maximum intensity minus the baseline at the apex), producing a zigzag index bounded in [0, 4.0]. Apply this as an EIC quality evaluation and filtering step, prior to chromatographic peak detection, so that 'bad' EICs (high zigzag values) are filtered out before the time-consuming peak detection processing. Decision points: compare against the MCQ index when the EIC has high background or a single peak — the article shows the MCQ gives misleadingly low scores in those cases while the zigzag index remains robust; EIC_ID 167 (bad) gets a low MCQ but a high zigzag, whereas EIC_IDs 27/90/134/154 (high-background or single-peak but locally good) get low MCQ values yet low zigzag values. For peaks, the zigzag index is one of six peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local zigzag) and is reported to outperform the others alongside sharpness and peak significance level due to the zigzag nature of LC/MS chromatographic peaks; combining several metrics is more efficient than any individual metric. Implementations described in the article were written in Matlab, as part of a four-module data processing program (acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering).

## Related tools

- **Matlab** (Programming language in which the authors' four-module data processing program (including EIC and peak quality evaluation/filtering) was implemented)
- **XCMS** (Open-source tool used to produce peak lists for ground truth definition in comprehensive evaluation of the metrics)
- **MZmine** (Open-source tool used to produce peak lists for ground truth definition in comprehensive evaluation of the metrics)
- **MAVEN** (Open-source tool used to produce peak lists for ground truth definition in comprehensive evaluation of the metrics)

## Evaluation signals

- The computed zigzag index must lie within [0, 4.0]; values outside this range indicate an implementation error
- For the representative bad EIC (EIC_ID 167), the zigzag index should be high while the MCQ index is low
- For high-background or single-peak but locally good EICs (EIC_IDs 27/90/134/154), the zigzag index should be low even where the MCQ index is low
- In comprehensive evaluation, varying the cutoff threshold should yield Recall, Precision, and F-Score values that can be tabulated; performance should be reported both for individual metrics and for combinations
- Ground truth peaks used for benchmarking should be defined as peaks detected by at least two of XCMS, MZmine, and MAVEN

## Limitations

- Setting an optimal cutoff threshold for each metric still is a challenge task
- Machine learning-based automatic peak identification (e.g. SVM) was not implemented; it would require compiling and curating a sufficient number of 'good' and 'bad' representative peaks for training samples
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository is provided, and no license is specified for usage
- The metric is EIC/peak-oriented for LC/MS-based metabolomics; it is not presented as a replacement for the MCQ index when selecting or aligning analyte components

## Evidence

- [abstract] Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak.: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [intro] Therefore, efficient methods to evaluate extracted EIC quality and filter out the "bad" EICs before the downstream time-consuming peak detection processing are highly desired.: "Therefore, efficient methods to evaluate extracted EIC quality and filter out the "bad" EICs before the downstream time-consuming peak detection processing are highly desired."
- [methods] which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering: "which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [intro] During data processing, the metric cutoff thresholds were varied, both individually and in combination,: "During data processing, the metric cutoff thresholds were varied, both individually and in combination,"
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [discussion] The LC/MS profile dataset and Matlab scripts are available upon request.: "The LC/MS profile dataset and Matlab scripts are available upon request."
