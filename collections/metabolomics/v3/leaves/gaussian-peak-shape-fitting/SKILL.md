---
name: gaussian-peak-shape-fitting
description: Use when comparing feature counts from LC-MS processing tools (asari,
  XCMS, MZmine) and you need an objective peak-quality metric to determine whether
  extra features are genuinely better coverage or low-quality peaks.
when_to_use_negative:
- When input is raw vendor data (e.g., Thermo .RAW) rather than centroided mzML plus
  an existing feature table — format conversion via ThermoRawFileParser or msConvert
  must happen first
- When working with low-resolution data, which asari's mass-track framework does not
  support
- When the goal is MS/MS identification or annotation rather than peak-quality assessment
  of detected features
tools:
- name: asari
  role: Provides the peaks.evaluate_gaussian_peak_on_intensity_list function for Gaussian
    peak-shape fitness scoring, and supplies the feature table (preferred_Feature_table.tsv)
    on which the scores are recomputed
  repo: https://github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Source of one of the comparison feature tables whose reported features are
    re-scored (1243 bad peaks on Yeast2021)
- name: MZmine
  role: Source of feature tables from two peak-detection algorithms (wavelets and
    local minimum) whose reported features are re-scored (4035 and 9436 bad peaks
    on Yeast2021)
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to centroided .mzML from which EIC intensity lists
    are extracted for peak-shape fitting
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:0ea72320b6b8291f7001fd7f75e2e3618bff798cbb5724148c947a92c3c0530b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - asari
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# gaussian-peak-shape-fitting

## Summary

Recompute a Gaussian peak-shape fitness score for each reported LC-MS feature by fitting a Gaussian model to the EIC intensity list over the peak region padded by 3 s on each side, then threshold at 0.5 to separate good-quality peaks from bad ones. This explains why different processing tools (asari, XCMS, MZmine) report very different feature counts: the surplus features are low-quality, non-reproducible peaks.

## When to use

Use when comparing feature counts from LC-MS processing tools (asari, XCMS, MZmine) and you need an objective peak-quality metric to determine whether extra features are genuinely better coverage or low-quality peaks.

## When NOT to use

- When input is raw vendor data (e.g., Thermo .RAW) rather than centroided mzML plus an existing feature table — format conversion via ThermoRawFileParser or msConvert must happen first
- When working with low-resolution data, which asari's mass-track framework does not support
- When the goal is MS/MS identification or annotation rather than peak-quality assessment of detected features

## Inputs

- Feature tables (e.g., asari preferred_Feature_table.tsv or export/full_Feature_table.tsv, plus XCMS and MZmine wavelet/local-minimum peak-detection tables) with reported peak m/z, RT boundaries, and sample intensities
- Extracted ion chromatogram (EIC) intensity lists over the reported peak region, derived from centroided mzML mass tracks
- Padding parameter (3 s on each side of the reported peak boundary)
- Fitness-score threshold (0.5)

## Outputs

- Per-feature Gaussian peak-shape fitness (goodness-of-fitting) scores
- Per-tool counts of bad peaks (fitness < 0.5) and good peaks (fitness > 0.5)
- Tabulation of feature counts and bad-peak ratios per tool, and a Figure 5b-style panel comparing good-peak numbers across the four tools

## How to apply

Load the feature tables produced by each tool (e.g., on the Yeast2021 dataset) and, for every reported feature, extract the EIC intensity list over the region bounded by the reported peak boundaries plus 3 s padding on the left and right. Fit a Gaussian model to each padded intensity list to obtain the peak-shape goodness-of-fitting score; asari implements this as the peaks.evaluate_gaussian_peak_on_intensity_list function. Flag features with fitness score below 0.5 as bad peaks and count them per tool, then compare good-peak (score > 0.5) counts across tools. The rationale is that good-peak counts are relatively stable across tools (~5000 on Yeast2021) while total feature counts diverge, so the divergence is attributable to low-quality, non-reproducible peaks rather than better coverage. When reproducing the published benchmark, the bad-peak counts must match 504 (asari), 1243 (XCMS), 4035 (MZmine wavelets) and 9436 (MZmine local minimum).

## Related tools

- **asari** (Provides the peaks.evaluate_gaussian_peak_on_intensity_list function for Gaussian peak-shape fitness scoring, and supplies the feature table (preferred_Feature_table.tsv) on which the scores are recomputed) — https://github.com/shuzhao-li-lab/asari
- **XCMS** (Source of one of the comparison feature tables whose reported features are re-scored (1243 bad peaks on Yeast2021))
- **MZmine** (Source of feature tables from two peak-detection algorithms (wavelets and local minimum) whose reported features are re-scored (4035 and 9436 bad peaks on Yeast2021))
- **ThermoRawFileParser** (Converts Thermo .RAW files to centroided .mzML from which EIC intensity lists are extracted for peak-shape fitting) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
from asari import peaks; fitness = peaks.evaluate_gaussian_peak_on_intensity_list(eic_intensity_padded_3s)
```

## Evaluation signals

- Recomputed bad-peak counts on Yeast2021 must match 504 (asari), 1243 (XCMS), 4035 (MZmine wavelets) and 9436 (MZmine local minimum)
- Good-peak counts (shape score > 0.5) should be stable across all four tools at close to 5000
- Every feature evaluated must have 3 s padding applied on both the left and right of its reported peak boundary before fitting
- Feature counts and bad-peak ratios per tool should tabulate consistently with the Figure 5b panel comparison
- All peaks retained in full_Feature_table.tsv should satisfy the stated signal (snr) and shape standards used by the source tool

## Limitations

- The wavelet peak-detection algorithm shows no advantage and adds unnecessary complexity, yet inflates bad-peak counts (4035 on Yeast2021)
- Local-minimum peak detection in MZmine yields a very high bad-peak count (9436), so tool-reported feature counts alone are misleading without quality control
- Borderline cases in feature filtering are a tradeoff between sensitivity and true discovery rate, so the 0.5 threshold should be interpreted with that tension in mind

## Evidence

- [methods] Gaussian fitness scoring function in asari peaks module: "[peaks.evaluate_gaussian_peak_on_intensity_list](peaks.evaluate_gaussian_peak_on_intensity_list)"
- [abstract] Tool inconsistency attributed to missing feature quality controls: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [discussion] Wavelet algorithm adds complexity without benefit: "The widely used wavelet algorithm shows no advantage in this study, but costs unnecessary complexity."
- [readme] asari tracks peak-quality metrics by design: "Tracking peak quality, selectiviy metrics on m/z, chromatography and annotation databases"
- [readme] Full feature table retains peaks meeting signal and shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
