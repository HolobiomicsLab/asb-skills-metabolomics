---
name: cross-tool-feature-comparison
description: Use when multiple LC-MS processing tools (asari, XCMS, MZmine with wavelet
  or local-minimum peak detection) report very different feature counts on the same
  dataset and you must determine whether the extra features are genuine or low-quality
  non-reproducible peaks.
when_to_use_negative:
- The data are low-resolution or from non-Orbitrap platforms without re-optimized
  settings, since the underlying processing tools were mostly tested on Orbitrap data
- The goal is tuning a single tool's parameters rather than a tool-independent, fair
  comparison of feature quality across tools
- Only one tool's feature table is available and no shared raw (mzML) data exist to
  extract comparable EICs across tools
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: asari
  role: Provides the Gaussian peak-shape fitness scoring function (peaks.evaluate_gaussian_peak_on_intensity_list)
    applied identically to all tools' features, and one of the compared feature tables
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Source of one compared feature table on the shared dataset
- name: MZmine
  role: Source of two compared feature tables (wavelet peak detection and local minimum
    peak detection)
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to centroided .mzML so all tools run on identical
    input
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:b6bddb676e234c38aaee39f1bb63a949cdd89259fc439e43a8d74aa726fc35b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-tool-feature-comparison

## Summary

Fairly compares LC-MS feature tables produced by asari, XCMS and MZmine by recomputing a tool-independent Gaussian peak-shape fitness score on each reported peak, showing that large differences in reported feature counts arise from low-quality peaks rather than better coverage.

## When to use

Use when multiple LC-MS processing tools (asari, XCMS, MZmine with wavelet or local-minimum peak detection) report very different feature counts on the same dataset and you must determine whether the extra features are genuine or low-quality non-reproducible peaks.

## When NOT to use

- The data are low-resolution or from non-Orbitrap platforms without re-optimized settings, since the underlying processing tools were mostly tested on Orbitrap data
- The goal is tuning a single tool's parameters rather than a tool-independent, fair comparison of feature quality across tools
- Only one tool's feature table is available and no shared raw (mzML) data exist to extract comparable EICs across tools

## Inputs

- Feature tables (feature lists with m/z, retention time, peak boundaries) from asari, XCMS, MZmine (wavelet) and MZmine (local minimum) run on the same LC-MS dataset
- Centroided mzML data files for the same acquisitions (to extract EIC intensity lists over padded peak regions)

## Outputs

- Per-tool bad-peak counts at fitness score < 0.5 (e.g. 504 asari, 1243 XCMS, 4035 MZmine wavelets, 9436 MZmine local minimum on Yeast2021)
- Per-tool good-peak counts (shape score > 0.5), expected to be stable at ~5000 across tools
- Table of feature counts and bad-peak ratios per tool; Figure 5b-style panel comparing good-peak numbers across the four tools

## How to apply

Load the feature tables from each tool run on the same dataset (e.g. Yeast2021). For every reported feature, extract the extracted-ion-chromatogram (EIC) intensity list over the peak region padded by 3 s on both the left and right of the reported peak boundaries — the same padding for all tools to avoid bias from boundary definitions. Fit a Gaussian model to each padded intensity list to obtain a peak-shape fitness (goodness-of-fitting) score per feature, using the same scoring function across tools (asari's peaks.evaluate_gaussian_peak_on_intensity_list implements this scoring). Flag features with fitness score below 0.5 as bad peaks and count bad peaks and good peaks per tool; on Yeast2021 this yields 504 bad peaks for asari, 1243 for XCMS, 4035 for MZmine (wavelets) and 9436 for MZmine (local minimum), while good-peak counts stay near 5000 across tools. Tabulate feature counts and bad-peak ratios per tool and plot the good-peak comparison (Figure 5b style); the rationale is that a tool-independent shape score controls for each tool's internal quality filters, attributing count inflation to low-quality, non-reproducible peaks.

## Related tools

- **asari** (Provides the Gaussian peak-shape fitness scoring function (peaks.evaluate_gaussian_peak_on_intensity_list) applied identically to all tools' features, and one of the compared feature tables) — https://github.com/shuzhao-li/asari
- **XCMS** (Source of one compared feature table on the shared dataset)
- **MZmine** (Source of two compared feature tables (wavelet peak detection and local minimum peak detection))
- **ThermoRawFileParser** (Converts Thermo .RAW files to centroided .mzML so all tools run on identical input) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
from scipy.optimize import curve_fit; popt, pcov = curve_fit(gaussian, x, eic_padded_3s); fitness = goodness_of_fit(eic_padded_3s, gaussian(x, *popt))
```

## Evaluation signals

- Recomputed bad-peak counts match the published values: 504 (asari), 1243 (XCMS), 4035 (MZmine wavelets), 9436 (MZmine local minimum) on Yeast2021
- Good-peak counts (fitness > 0.5) are stable across all four tools at close to 5000
- EIC extraction uses exactly 3 s padding on both sides of each reported peak boundary, identical across tools
- The same Gaussian fitting function and 0.5 fitness threshold are applied uniformly to every tool's features
- Feature counts and bad-peak ratios are tabulated per tool so count inflation is attributable to low-quality peaks

## Limitations

- The widely used wavelet algorithm shows no advantage in this study but adds unnecessary complexity (MZmine wavelets)
- Inconsistency among tools is attributed to deficiencies in mass alignment and controls of feature quality, so discrepancies may also reflect alignment errors, not only peak shape
- Borderline cases are a tradeoff between sensitivity and true discovery rate; the 0.5 fitness cutoff is a fixed quality gate

## Evidence

- [abstract] Inconsistency among tools attributed to mass alignment and feature quality controls: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [discussion] Wavelet algorithm shows no advantage: "The widely used wavelet algorithm shows no advantage in this study, but costs unnecessary complexity."
- [methods] Gaussian scoring function on intensity lists: "[peaks.evaluate_gaussian_peak_on_intensity_list](peaks.evaluate_gaussian_peak_on_intensity_list)"
- [readme] Feature tables retain peaks meeting shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
