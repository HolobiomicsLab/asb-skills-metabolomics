---
name: metric-robustness-comparison-across-chromatogram-scenarios
description: Use when evaluating whether a chromatogram quality metric (e.g. MCQ,
  zigzag index, sharpness, SNR) is fair across EICs with high background signals,
  single-peak EICs, or zigzag-contaminated profiles, and you need an evidence-based
  robustness verdict per scenario.
when_to_use_negative:
- When the input is raw centroided profile data not yet reduced to extracted ion chromatograms
  — EIC extraction and peak detection are prerequisites handled by a different module
- When you only need an absolute quality cutoff for filtering a large peak list (e.g.
  cutoff threshold tuning for Recall/Precision/F-Score) rather than a cross-scenario
  robustness comparison of metric definitions
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Implementation environment for the custom data-processing program and quality-metric
    computations
- name: XCMS
  role: One of the open-source peak-detection tools used to generate the ground-truth
    peak list for comprehensive evaluation
- name: MZmine
  role: One of the open-source peak-detection tools used to generate the ground-truth
    peak list for comprehensive evaluation
- name: MAVEN
  role: One of the open-source peak-detection tools used to generate the ground-truth
    peak list for comprehensive evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:0f28fe03b905b0640f544402acb95fed325f5369b82b93b987b26e0dd6275511
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metric-robustness-comparison-across-chromatogram-scenarios

## Summary

Compare candidate chromatogram/peak quality metrics side-by-side on representative EICs spanning easy and hard scenarios (high background, single peak, zigzag noise) to establish which metric is robust across all of them. Grounded in the MCQ vs. global zigzag index contrast from Zhang & Zhao's LC/MS metabolomics quality-evaluation study.

## When to use

Use when evaluating whether a chromatogram quality metric (e.g. MCQ, zigzag index, sharpness, SNR) is fair across EICs with high background signals, single-peak EICs, or zigzag-contaminated profiles, and you need an evidence-based robustness verdict per scenario.

## When NOT to use

- When the input is raw centroided profile data not yet reduced to extracted ion chromatograms — EIC extraction and peak detection are prerequisites handled by a different module
- When you only need an absolute quality cutoff for filtering a large peak list (e.g. cutoff threshold tuning for Recall/Precision/F-Score) rather than a cross-scenario robustness comparison of metric definitions

## Inputs

- Raw LC/MS profile data (e.g. from an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer) from which EICs are extracted
- Extracted ion chromatogram (EIC) intensity profiles as numeric vectors, with known good/bad labels for representative EICs (e.g. Table 1 EIC_IDs)
- Metric implementations for MCQ index, global zigzag index, and optionally sharpness, Gaussian similarity, SNR, peak significance level, TPASR

## Outputs

- Per-EIC matrix of metric values (e.g. MCQ index, global zigzag index) for each representative EIC
- Qualitative good/bad verdict per EIC under each metric, exposing scenario-specific robustness or unfairness
- Summary table establishing which metrics are robust across high-background, single-peak, and zigzag scenarios (as in Table 1)

## How to apply

Select a small, curated set of representative EIC intensity profiles (e.g. EIC_IDs 27, 40, 90, 134, 154, 167 from the 611 extracted EICs with known good/bad labels). For each EIC compute every candidate metric with its exact definition: MCQ index as similarity of the original mass chromatogram to its smoothed version (spike detection) and to its mean-subtracted version (background detection); the global zigzag index via first-order derivatives as sum of (2*In - In-1 - In+1)^2 over neighboring data points, normalized by N times the squared effective peak intensity (max minus baseline at apex), yielding a value in [0, 4.0]. Tabulate per-EIC metric values alongside the qualitative good/bad verdict under each metric, then judge robustness by whether a metric consistently classifies the known bad cases (e.g. EIC_ID 167 should get low MCQ and high zigzag) without penalizing the locally good high-background or single-peak EICs (e.g. EIC_IDs 27, 90, 134, 154 should get low zigzag despite low MCQ). In this article's contrast, the MCQ index could not fairly evaluate EICs with high background signals or with only a single peak, while the EIC-related global zigzag index efficiently evaluated both scenarios. Where the article's peak-metric comparison was also run (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag), sharpness, peak significance level, and zigzag index outperformed the others due to the zigzag nature of LC/MS chromatographic peaks.

## Related tools

- **Matlab** (Implementation environment for the custom data-processing program and quality-metric computations)
- **XCMS** (One of the open-source peak-detection tools used to generate the ground-truth peak list for comprehensive evaluation)
- **MZmine** (One of the open-source peak-detection tools used to generate the ground-truth peak list for comprehensive evaluation)
- **MAVEN** (One of the open-source peak-detection tools used to generate the ground-truth peak list for comprehensive evaluation)

## Examples

```
% MATLAB: compute MCQ index and global zigzag index for a representative EIC vector y (1xN double)
mcq = mcqIndex(y, smoothed_y, mean_subtracted_y); % spike & background similarity
zz = sum((2*y(2:end-1) - y(1:end-2) - y(3:end)).^2) / (length(y) * (max(y)-baseline_at_apex)^2); % in [0,4.0]
```

## Evaluation signals

- For a known bad EIC (e.g. EIC_ID 167), the metric values must reproduce the expected qualitative pattern (low MCQ, high zigzag index); if not, the metric implementation is likely wrong
- For high-background or single-peak EICs that are locally good (e.g. EIC_IDs 27, 90, 134, 154), a robust metric must yield low (good) values — a metric that gives high (bad) values here is unfair to the scenario
- The global zigzag index must remain bounded in [0, 4.0]; values outside this range indicate an implementation error in the normalization
- Metric values should be reproducible across the 611 extracted EICs when re-run on the same LC/MS profile dataset
- A fair comparison must include all metrics under identical EIC inputs and cutoff logic; discrepancies in scenario classification across metrics should be explicitly tabulated

## Limitations

- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak, so relying on MCQ alone would mislabel these scenarios
- Setting an optimal cutoff threshold for each metric still is a challenge; results depend on chosen thresholds
- Machine learning-based automatic peak identification (e.g. SVM) is not yet implemented because a sufficient curated training set of good and bad representative peaks does not yet exist
- Robustness conclusions were drawn on representative EICs from a specific LC/MS profile dataset; generalization to other instruments or acquisition methods is not demonstrated here

## Evidence

- [abstract] Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak.: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [intro] We conducted both case studies and comprehensive performance evaluations of these metrics. ... During data processing, the metric cutoff thresholds were varied, both individually and in combination,: [evidence span withheld — anchor status: not_found]
- [methods] a real LC/MS profile dataset was generated using an Agilent HPLC system interfaced with a quadrupole-time-of-flight (Q-TOF) Premier mass spectrometer: "a real LC/MS profile dataset was generated using an Agilent HPLC system interfaced with a quadrupole-time-of-flight (Q-TOF) Premier mass spectrometer"
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: "we developed our own data processing program in Matlab, which consists of four sequential modules"
