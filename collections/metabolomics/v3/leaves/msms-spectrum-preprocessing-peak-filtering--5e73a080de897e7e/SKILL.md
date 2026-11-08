---
name: msms-spectrum-preprocessing-peak-filtering
description: Use when raw MS/MS peak files (mzML, mzXML, or MGF) must be cleaned before
  spectrum similarity computation, feature hashing, or clustering — e.g. precursor
  peaks and noise dominate the spectrum and would distort cosine similarity.
when_to_use_negative:
- Input spectra are metabolomics or top-down data with few, widely spaced peaks —
  the default min_peaks=5 and 250 m/z range filters would incorrectly discard valid
  spectra
- Input is already a processed/consensus spectrum (e.g. cluster representative from
  a prior falcon run) where re-filtering would remove informative peaks
- The downstream similarity measure requires raw untransformed intensities and you
  have not adjusted the eps cosine threshold for the scaling method
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Implements the preprocessing stage (precursor/noise peak removal, min_peaks/min_mz_range/min_mz/max_mz
    filters, intensity scaling) prior to feature hashing and clustering
  repo: https://github.com/bittremieux/falcon
- name: spectrum_utils
  role: Used for spectrum preprocessing operations such as removing precursor peaks
    and applying the square-root intensity transform (pinned to 0.3.5 for falcon)
- name: Pyteomics
  role: Loads MS/MS spectra from MGF peak files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:be67650d4ac50cb405c4482cd07818e08d018226a933c941632821abbc321728
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msms-spectrum-preprocessing-peak-filtering

## Summary

Preprocess high-resolution MS/MS spectra prior to clustering or similarity search by removing precursor-ion and low-intensity noise peaks, restricting to the most intense peaks, discarding low-quality spectra, and applying a square-root intensity transformation to de-emphasize dominant peaks.

## When to use

Use when raw MS/MS peak files (mzML, mzXML, or MGF) must be cleaned before spectrum similarity computation, feature hashing, or clustering — e.g. precursor peaks and noise dominate the spectrum and would distort cosine similarity.

## When NOT to use

- Input spectra are metabolomics or top-down data with few, widely spaced peaks — the default min_peaks=5 and 250 m/z range filters would incorrectly discard valid spectra
- Input is already a processed/consensus spectrum (e.g. cluster representative from a prior falcon run) where re-filtering would remove informative peaks
- The downstream similarity measure requires raw untransformed intensities and you have not adjusted the eps cosine threshold for the scaling method

## Inputs

- MS/MS peak files in mzML, mzXML, or MGF format
- Precursor m/z and charge information per spectrum
- falcon preprocessing configuration (min_peaks, min_mz_range, min_mz, max_mz, scaling)

## Outputs

- Filtered, intensity-scaled spectra restricted to at most 50 peaks
- Preprocessed spectra ready for 0.05 m/z binned sparse vectorization and feature hashing
- Discard report/labels for low-quality spectra (< 5 peaks or m/z range < 250 Da)

## How to apply

Load spectra from MGF (or mzML/mzXML) peak files, then remove peaks corresponding to the precursor ion and low-intensity noise peaks (in the falcon pipeline, peaks below 10% of the base peak intensity), restricting each spectrum to its 50 most intense peaks. Discard low-quality spectra that have fewer than five peaks remaining or whose mass range between the minimum and maximum peak m/z is less than 250 Da; configurable min_mz/max_mz bounds (defaults 101 and 500 m/z) can further trim peaks outside the analyte range. Square-root transform the peak intensities to de-emphasize overly dominant peaks, since this scaling method 'often gives good results' for proteomics data. Note that the scaling choice interacts with downstream similarity thresholds — the README warns that the scaling method can influence the cosine distance threshold eps used in DBSCAN clustering — so parameters must be adjusted together. For metabolomics or top-down data the defaults (min 5 peaks, 250 m/z range) should be reduced.

## Related tools

- **falcon** (Implements the preprocessing stage (precursor/noise peak removal, min_peaks/min_mz_range/min_mz/max_mz filters, intensity scaling) prior to feature hashing and clustering) — https://github.com/bittremieux/falcon
- **spectrum_utils** (Used for spectrum preprocessing operations such as removing precursor peaks and applying the square-root intensity transform (pinned to 0.3.5 for falcon))
- **Pyteomics** (Loads MS/MS spectra from MGF peak files)

## Examples

```
falcon peak/*.mzml falcon --min_peaks 5 --min_mz_range 250 --min_mz 101 --max_mz 500 --scaling sqrt --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Every retained spectrum has at most 50 peaks and at least 5 peaks after filtering
- Every retained spectrum spans an m/z range of at least 250 Da between its minimum and maximum peak
- Precursor-ion peaks are absent from the filtered peak lists
- Peak intensities are square-root transformed, and no peak falls below the noise cutoff (peaks below ~10% of base peak intensity removed)
- Filtered spectra feed vectorization of shape (n_spectra, 800) with finite values after feature hashing, with norms approximately conserved

## Limitations

- The default settings are intended for bottom-up proteomics; metabolomics and top-down data likely need reduced min_peaks/min_mz_range and adjusted min_mz/max_mz values
- The intensity scaling method influences the cosine threshold eps, so changing preprocessing without retuning eps degrades cluster purity
- The influence of MS/MS spectrum preprocessing on various similarity measures has not been systematically evaluated (noted as an open question in the paper's discussion)

## Evidence
