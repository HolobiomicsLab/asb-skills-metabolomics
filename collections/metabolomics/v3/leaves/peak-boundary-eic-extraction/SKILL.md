---
name: peak-boundary-eic-extraction
description: Use when comparing feature tables from LC-MS processing tools and you
  must judge whether extra reported features are genuine or low-quality, by recomputing
  a shape-fitness score on the EIC around each reported peak boundary.
when_to_use_negative:
- The goal is de novo peak detection rather than scoring the quality of already-reported
  peak boundaries
- Data are low-resolution or from very high-noise acquisitions — the asari framework
  does not work with low-resolution data and high-noise data may need customized processing
- You need MS/MS handling — the default LC-MS workflow in asari ignores MS/MS spectra,
  so shape scoring applies only to MS1 features
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: asari
  role: Source of mass tracks / EIC data and the Gaussian peak-shape evaluation function
    used to score fitness on padded intensity lists
  repo: https://github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Feature table whose reported peak boundaries are re-evaluated by Gaussian
    fitness scoring
- name: MZmine
  role: Feature tables from both wavelet and local-minimum peak detection whose reported
    peak boundaries are re-evaluated by Gaussian fitness scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:5e55298a8d0c5f6ada162e4d99e711b2b68d87cabcff06f72647468ce17dd7e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/asari
  tools:
  - MZmine
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-boundary-eic-extraction

## Summary

Extracts EIC intensity lists over reported LC-MS peak boundaries (padded by 3 s on each side) and fits a Gaussian to compute a peak-shape fitness score, used to separate good-quality from bad-quality features across processing tools (asari, XCMS, MZmine).

## When to use

Use when comparing feature tables from LC-MS processing tools and you must judge whether extra reported features are genuine or low-quality, by recomputing a shape-fitness score on the EIC around each reported peak boundary.

## When NOT to use

- The goal is de novo peak detection rather than scoring the quality of already-reported peak boundaries
- Data are low-resolution or from very high-noise acquisitions — the asari framework does not work with low-resolution data and high-noise data may need customized processing
- You need MS/MS handling — the default LC-MS workflow in asari ignores MS/MS spectra, so shape scoring applies only to MS1 features

## Inputs

- Feature tables with m/z, retention time and peak start/end boundaries per feature (asari preferred_Feature_table.tsv, XCMS, MZmine wavelet, MZmine local-minimum outputs)
- Centroided mzML files (or precomputed EIC/mass-track data) covering the retention-time range of each reported peak

## Outputs

- Per-feature Gaussian peak-shape fitness (goodness-of-fitting) scores
- Per-tool bad-peak counts (fitness < 0.5), e.g. 504 asari, 1243 XCMS, 4035 MZmine wavelets, 9436 MZmine local minimum on Yeast2021
- Comparison figure/panel of good-peak numbers (fitness > 0.5) across the four tools

## How to apply

Load the feature tables (e.g. from asari, XCMS, MZmine wavelet and MZmine local-minimum on Yeast2021) and take each feature's reported m/z and peak start/end boundaries. For each feature, pull the EIC intensity list over the peak region with 3 s padding added on both the left and right of the reported boundary (asari mass tracks / EICs support this extraction via functions like extract_single_track_fullrt_length). Fit a Gaussian model to the padded intensity list using the article's peak-evaluation function (peaks.evaluate_gaussian_peak_on_intensity_list) to obtain a goodness-of-fitting score per feature. Flag features with fitness < 0.5 as bad peaks and count them per tool; the rationale (from the asari paper) is that inconsistencies between tools arise from low-quality, non-reproducible peaks rather than better coverage, since good-peak counts stay stable (~5000 on Yeast2021) while bad-peak counts diverge sharply.

## Related tools

- **asari** (Source of mass tracks / EIC data and the Gaussian peak-shape evaluation function used to score fitness on padded intensity lists) — https://github.com/shuzhao-li-lab/asari
- **XCMS** (Feature table whose reported peak boundaries are re-evaluated by Gaussian fitness scoring)
- **MZmine** (Feature tables from both wavelet and local-minimum peak detection whose reported peak boundaries are re-evaluated by Gaussian fitness scoring)

## Examples

```
from asari import peaks; fitness = peaks.evaluate_gaussian_peak_on_intensity_list(intensity_list_padded_3s)
```

## Evaluation signals

- Recomputed bad-peak counts (fitness < 0.5) match published values: 504 (asari), 1243 (XCMS), 4035 (MZmine wavelets), 9436 (MZmine local minimum) on Yeast2021
- Good-peak counts (fitness > 0.5) are stable across all four tools at approximately 5000
- Every scored feature was evaluated on an intensity list padded exactly 3 s on both left and right of its reported peak boundary
- Per-tool feature counts and bad-peak ratios are tabulated and the good-peak comparison panel (Figure 5b-like) is reproducible

## Limitations

- The asari settings underlying EIC/shape evaluation were mostly tested on Orbitrap platforms and are not yet optimized for other instruments
- The software does not work with low-resolution data, which limits where this shape-scoring approach applies
- ppm tolerance is only a practical approximation; a 2nd order polynomial model may be required for the full m/z range
- The widely used wavelet algorithm shows no advantage in this study but costs unnecessary complexity, so wavelet-based MZmine detection adds features that score as low-quality

## Evidence

- [abstract] Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality: "Inconsistency among the tools examined is attributed to the deficiencies of mass alignment and controls of feature quality"
- [intro] Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7.: "Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7."
- [methods] peaks.evaluate_gaussian_peak_on_intensity_list: "[peaks.evaluate_gaussian_peak_on_intensity_list](peaks.evaluate_gaussian_peak_on_intensity_list)"
- [discussion] The widely used wavelet algorithm shows no advantage in this study, but costs unnecessary complexity.: "The widely used wavelet algorithm shows no advantage in this study, but costs unnecessary complexity."
- [readme] Reproducible, track and backtrack between features and mass tracks (EICs): "Reproducible, track and backtrack between features and mass tracks (EICs)"
- [readme] All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards: "All peaks are kept in `export/full_Feature_table.tsv` if they meet signal (snr) and shape standards"
