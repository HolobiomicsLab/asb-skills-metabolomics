---
name: peak-quality-threshold-counting
description: Use when comparing feature tables from multiple LC-MS processing tools
  (e.g. asari, XCMS, MZmine) on the same raw data and you need to determine whether
  differences in reported feature counts reflect genuine coverage or low-quality peaks.
when_to_use_negative:
- The goal is metabolite annotation or identification — this skill only quantifies
  peak-shape quality and does not assign compounds
- 'Data are low-resolution MS: asari''s mass-track-based processing and this quality
  metric assume high-resolution data'
- A single tool's feature table is being validated with the tool's own internal QC
  rather than a cross-tool feature-count comparison
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Source of the Gaussian peak-shape fitness scoring (peaks.evaluate_gaussian_peak_on_intensity_list)
    and one of the four compared feature tables
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Produces a compared feature table whose extra reported features are evaluated
    as bad peaks under the recomputed fitness scores
- name: MZmine
  role: Produces two compared feature tables (wavelet and local minimum peak detection)
    evaluated for bad-peak counts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:6efe3b7932555e85f0f05f1ee0d66035d8702fea8f8dd3b91525154d3ab704af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - MZmine
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-quality-threshold-counting

## Summary

Counts good- vs bad-quality LC-MS features by fitting a Gaussian to each extracted ion chromatogram (EIC, padded 3 s on both sides of the reported peak boundary) and thresholding the peak-shape fitness score at 0.5. Used to show that inflated feature counts from tools like XCMS and MZmine arise mostly from low-quality, non-reproducible peaks rather than better coverage.

## When to use

Use when comparing feature tables from multiple LC-MS processing tools (e.g. asari, XCMS, MZmine) on the same raw data and you need to determine whether differences in reported feature counts reflect genuine coverage or low-quality peaks.

## When NOT to use

- The goal is metabolite annotation or identification — this skill only quantifies peak-shape quality and does not assign compounds
- Data are low-resolution MS: asari's mass-track-based processing and this quality metric assume high-resolution data
- A single tool's feature table is being validated with the tool's own internal QC rather than a cross-tool feature-count comparison

## Inputs

- Feature tables (m/z, retention time, peak boundaries) from asari, XCMS, and MZmine (wavelet and local minimum peak detection) run on the same LC-MS dataset
- Raw centroided mzML data (or mass tracks/EICs) for the same samples to extract intensity lists over peak regions
- Peak-shape fitness threshold (0.5) and EIC padding (3 s per side)

## Outputs

- Per-tool counts of bad peaks (Gaussian fitness score < 0.5) and good peaks (score > 0.5)
- A table of feature counts and bad-peak ratios per tool (e.g. asari 504, XCMS 1243, MZmine wavelets 4035, MZmine local minimum 9436 bad peaks on Yeast2021)
- A Figure 5b-style comparison panel showing stable ~5000 good peaks per tool against diverging total feature counts

## How to apply

Load the feature tables produced by each tool on the same dataset (here Yeast2021, processed by asari, XCMS, MZmine wavelet peak detection and MZmine local minimum peak detection). For each reported feature, extract the EIC intensity list over the peak region padded by 3 s on the left and right of the reported peak boundaries, then fit a Gaussian model to each padded intensity list to obtain a peak-shape fitness (goodness-of-fitting) score — the same scoring machinery as asari's peaks.evaluate_gaussian_peak_on_intensity_list. Flag features with fitness score below 0.5 as bad peaks and count them per tool; tabulate feature counts and bad-peak ratios per tool. Finally compare good-peak (score > 0.5) counts across tools: if they are stable while total counts diverge, the excess features are low-quality, non-reproducible peaks.

## Related tools

- **asari** (Source of the Gaussian peak-shape fitness scoring (peaks.evaluate_gaussian_peak_on_intensity_list) and one of the four compared feature tables) — https://github.com/shuzhao-li/asari
- **XCMS** (Produces a compared feature table whose extra reported features are evaluated as bad peaks under the recomputed fitness scores)
- **MZmine** (Produces two compared feature tables (wavelet and local minimum peak detection) evaluated for bad-peak counts)

## Examples

```
from asari import peaks; score = peaks.evaluate_gaussian_peak_on_intensity_list(eic_intensities_padded_3s)
```

## Evaluation signals

- Recomputed bad-peak counts must match published values: 504 (asari), 1243 (XCMS), 4035 (MZmine wavelets), 9436 (MZmine local minimum) on Yeast2021
- Good-peak counts (fitness > 0.5) should be stable at approximately 5000 across all four tools
- Bad-peak ratio should increase with total feature count (asari lowest, MZmine local minimum highest)
- Fitness scores must be computed on intensity lists extracted with 3 s padding on each side of the reported peak boundary — omitting padding changes scores and counts
- Every feature in each input table receives a finite fitness score; no feature should be dropped before scoring

## Limitations

- The 0.5 fitness cutoff and 3 s padding are dataset-tuned choices; results depend on chromatographic gradient length and instrument platform
- Results are platform-specific: asari is mostly tested on Orbitrap platforms and its settings need optimization for others
- The wavelet algorithm showed no advantage in this study but adds unnecessary complexity, so wavelet-based counts should be interpreted with that caveat
- Borderline peak-quality cases are a tradeoff between sensitivity and true discovery rate

## Evidence

- [results] Gaussian fitness threshold and padding from task card: [evidence span withheld — anchor status: not_found]
- [results] Good-peak stability finding: [evidence span withheld — anchor status: not_found]
- [results] Interpretation of excess features: [evidence span withheld — anchor status: not_found]
- [methods] Asari's Gaussian scoring function: "[peaks.evaluate_gaussian_peak_on_intensity_list](peaks.evaluate_gaussian_peak_on_intensity_list)"
- [abstract] Tool inconsistency attribution: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [discussion] Wavelet caveat: "The widely used wavelet algorithm shows no advantage in this study, but costs unnecessary complexity."
