---
name: spectral-quality-filtering
description: Use when raw MS/MS peak files (mzML, mzXML, or MGF) must be cleaned before
  spectrum clustering or similarity searching, and spectra fail quality criteria such
  as peak count, m/z range, or dominant noise peaks.
when_to_use_negative:
- Analyzing metabolomics or top-down data with bottom-up proteomics defaults — the
  README states the default min_peaks (5) and min_mz_range (250 m/z) 'likely need
  to be adjusted accordingly' and recommends reducing these values for metabolomics
  data
- Input spectra are already cluster consensus/representative spectra that have already
  been quality-filtered
- The downstream similarity measure requires untransformed absolute intensities, since
  square root (or other) scaling changes intensity ratios
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Implements spectrum preprocessing (precursor-ion and noise peak removal, min_peaks/min_mz_range/min_mz/max_mz
    filters, intensity scaling) prior to feature hashing and clustering
  repo: https://github.com/bittremieux/falcon
- name: spectrum_utils
  role: Used for spectrum preprocessing such as square-root transformation of peak
    intensities (pinned as spectrum-utils==0.3.5)
  repo: https://github.com/bittremieux/falcon
- name: Pyteomics
  role: Used to load MS/MS spectra from MGF peak files before filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:7acefd2196511bea400df38a1dc2fd2ae4f94c80d678842c2dcec5e7709fe5cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - NumPy
  - Numba
  - Pyteomics
  - Python
  - SciPy
  - falcon
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# spectral-quality-filtering

## Summary

A spectrum preprocessing stage that removes precursor-ion and low-intensity noise peaks, restricts each MS/MS spectrum to its 50 most intense peaks, discards low-quality spectra, and transforms peak intensities prior to clustering, so that downstream similarity measures operate on clean, comparable spectra.

## When to use

Use when raw MS/MS peak files (mzML, mzXML, or MGF) must be cleaned before spectrum clustering or similarity searching, and spectra fail quality criteria such as peak count, m/z range, or dominant noise peaks.

## When NOT to use

- Analyzing metabolomics or top-down data with bottom-up proteomics defaults — the README states the default min_peaks (5) and min_mz_range (250 m/z) 'likely need to be adjusted accordingly' and recommends reducing these values for metabolomics data
- Input spectra are already cluster consensus/representative spectra that have already been quality-filtered
- The downstream similarity measure requires untransformed absolute intensities, since square root (or other) scaling changes intensity ratios

## Inputs

- MS/MS spectra from peak files in mzML, mzXML, or MGF format (e.g. loaded via Pyteomics)
- falcon preprocessing configuration (min_peaks, min_mz_range, min_mz, max_mz, scaling)

## Outputs

- Filtered and intensity-transformed spectra containing only high-quality, high-intensity peaks retained for feature-hashing and nearest neighbor indexing
- Low-quality spectra removed from the clustering pipeline (fewer than 5 peaks or m/z range < 250)

## How to apply

Preprocess each spectrum by first removing peaks corresponding to the precursor ion and low-intensity noise peaks (peaks below 10% of the base peak intensity), then restricting each spectrum to its 50 most intense peaks. Discard low-quality spectra that have fewer than five peaks remaining (min_peaks=5) or whose m/z range between the minimum and maximum peak is less than 250 m/z (min_mz_range=250); peaks outside configured min_mz/max_mz bounds (default 101–500 m/z for bottom-up proteomics) are also discarded. Optionally scale peak intensities (square root, logarithm, rank, or none) to de-emphasize overly dominant peaks — square root scaling often gives good results, but note that the scaling method influences the cosine distance threshold eps used downstream. When clustering metabolomics or top-down data rather than bottom-up proteomics data, reduce min_peaks and min_mz_range accordingly.

## Related tools

- **falcon** (Implements spectrum preprocessing (precursor-ion and noise peak removal, min_peaks/min_mz_range/min_mz/max_mz filters, intensity scaling) prior to feature hashing and clustering) — https://github.com/bittremieux/falcon
- **spectrum_utils** (Used for spectrum preprocessing such as square-root transformation of peak intensities (pinned as spectrum-utils==0.3.5)) — https://github.com/bittremieux/falcon
- **Pyteomics** (Used to load MS/MS spectra from MGF peak files before filtering)

## Examples

```
falcon peak/*.mzml falcon --min_peaks 5 --min_mz_range 250 --min_mz 101 --max_mz 500 --scaling sqrt
```

## Evaluation signals

- Filtered spectra retain no precursor-ion peaks and no peaks below 10% of the base peak intensity
- No spectrum entering downstream hashing has fewer than 5 peaks (min_peaks) or an m/z span below 250 between its minimum and maximum peak (min_mz_range)
- Each retained spectrum contains at most its 50 most intense peaks
- Square-root-transformed spectra remain finite-valued and norm-consistent so cosine distance approximated after feature hashing matches the original spectra
- Changing the scaling method should prompt re-tuning the eps cosine distance threshold (e.g. values between 0.05 and 0.15 typically yield a pure clustering)

## Limitations

- Default preprocessing settings are intended for bottom-up proteomics; metabolomics and top-down data likely need adjusted min_peaks and min_mz_range
- The article's discussion notes that the influence of MS/MS spectrum preprocessing on various similarity measures was not evaluated
- The scaling method influences the cosine threshold eps, so changing it invalidates previously tuned clustering thresholds

## Evidence
